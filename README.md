# Hello World Module
## A Helm Chart for an example Mesh for Data module

## Introduction

This helm chart defines a common structure to deploy a Kubernetes job for an M4D module.

The configuration for the chart is in the values file.

## Prerequisites

- Kubernetes cluster 1.10+
- Helm 3.0.0+

## Installation

### Modify values in Makefile

In `Makefile`:
- Change `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `DOCKER_HOSTNAME`, `DOCKER_NAMESPACE`, `DOCKER_TAGNAME`, `DOCKER_IMG_NAME`, and `DOCKER_CHART_IMG_NAME` to your own preferences.

### Build Docker image for Python application
```bash
make docker-build
```

### Push Docker image to your preferred container registry
```bash
make docker-push
```

### Configure the chart

- Settings can configured by editing the `values.yaml` directly.
- Modify repository in `values.yaml` to your preferred Docker image. 
- Modify copy/read action as needed with appropriate values. 

### Login to Helm registry
```bash
make helm-login
```

### Lint and install Helm chart
```bash
make helm-verify
```

### Push the Helm chart

```bash
make helm-chart-push
```

## Uninstallation
```bash
make helm-uninstall
```

## Deploy M4D module
1. In your module yaml spec (`hello-world-module.yaml`):
    * Change `spec.chart.name` to your preferred chart image.
    * Define `flows` and `capabilities` for your module. 
    * The Mesh for Data manager checks the `statusIndicators` provided to see if the module is ready. In this example, if the Kubernetes job completes, the status will be `succeeded` and the manager will set the module as ready. 

2. Deploy `M4DModule` in `m4d-system` namespace:
```bash
kubectl create -f hello-world-module.yaml -n m4d-system
```
## Register data asset in Egeria and S3 bucket credentials in Vault (optional)
1. Follow steps 3 and 4 in [this example](https://ibm.github.io/the-mesh-for-data/docs/usage/notebook-sample/) to register the data asset in the catalog and set the `ASSET_ID` environment variable
2. Follow step 5 in [this example](https://ibm.github.io/the-mesh-for-data/docs/usage/notebook-sample/) to register HMAC credentials in Vault

## Deploy M4D application which triggers module
1. In `m4dapplication.yaml`:
    * Change `metadata.name` to your application name.
    * Define `appInfo.purpose`, `appInfo.role`, and `spec.data`
    * This ensures that a copy is triggered:
    ```yaml
    copy:
      required:true
    ```
2.  Deploy `M4DApplication` in `default` namespace:
```bash
cat m4dapplication.yaml | sed "s/ASSET_ID/$ASSET_ID/g" | kubectl -n default apply -f -
```
3.  Check if `M4DApplication` successfully deployed:
```bash
kubectl get m4dapplication -n default
kubectl describe M4DApplication hello-world-module-test -n default
```

4.  Check if module was triggered in `m4d-blueprints`:
```bash
kubectl get blueprint -n m4d-blueprints
kubectl describe blueprint hello-world-module-test-default -n m4d-blueprints
kubectl get job -n m4d-blueprints
kubectl get pods -n m4d-blueprints
```
If you are using the `hello-world-module` image, you should see this in the `kubectl logs` of your completed Pod:
```
$ kubectl logs rel1-hello-world-module-x2tgs

Hello World Module!

Connection name is s3

Connection format is parquet

Vault credential address is http://vault.m4d-system:8200/

Vault credential role is module

Vault credential secret path is v1/m4d/dataset-creds/%7B%22asset_id%22:%20%225067b64a-67bc-4067-9117-0aff0a9963ea%22%2C%20%22catalog_id%22:%20%220fd6ff25-7327-4b55-8ff2-56cc1c934824%22%7D

S3 bucket is m4d-test-bucket

S3 endpoint is s3.eu-gb.cloud-object-storage.appdomain.cloud

COPY SUCCEEDED
```


