---
title: Installing and Configuring Pivotal Build Service
owner: Partners
---

<strong><%= modified_date %></strong>

This topic describes how to install and configure Pivotal Build Service.

## <a id='install'></a> Install and Configure Pivotal Build Service

## Prerequisites

- PKS installed
- Kubectl installed locally (Only required if no ingress controller is already installed)
- Ruby (This is required to create the UAA client)


## Retrieve cluster credentials

This step retrieves the credentials used by `kubectl` to talk to the
PKS cluster where Pivotal Build Service will run

```bash
pks get-credentials <cluster-name>
```

To target the cluster execute

```bash
kubectl config use-context <cluster-name>
```

These commands only needs to be run once for the full installation.

## <a id='uaa-client-creation'></a> Install UAA Pivotal Build Service client

### Prerequisites
- Ruby

### Install the UAA client

The users of Pivotal Build Service are configured on a UAA.
In order to talk with UAA, Pivotal Build Service must have a client
configured. To configure this client we recommend using `uaac` tool

1. Install `uaac` tool on your machine, run the following command

  ```bash
  gem install cf-uaac
  ```

  **Note:** if not using `rbenv` or `rvm` you may need to execute `sudo gem install cf-uaac`

1. Target the UAA that will be used to authenticate the Build Service Users

  ```bash
  uaac target <UAA_URL>
  ```

  **Note:** When using a self-signed certificate, you must use the `--skip-ssl-validation` flag in conjuction with `uaac` 

1. Install the UAA Client

  ```bash
  uaac client add pb_cli --scope="openid" --secret="" --authorized_grant_types="password,refresh_token" --access_token_validity 600 --refresh_token_validity 21600
  ```
  
  **Note:** this command need to be executed as is. The secret in this case **need** to be an empty string



## Configure TLS certificates for Pivotal Build Service

Download the `.crt` and `.key` files to `/tmp/certificate.crt` and `/tmp/certificate.key`

Create the secret in the Kubernetes cluster

```bash
tlsCert=$(cat /tmp/certificate.crt | base64)
tlsKey=$(cat /tmp/certificate.key | base64)
cat << EOF| kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: build-service-certificate
  namespace: default
data:
  tls.crt: $tlsCert
  tls.key: $tlsKey
type: kubernetes.io/tls
EOF
```

After this step the files can be removed.

**NOTES:** For MacOS, when using `pb` cli the CA certificate should be added to the keychain and the Trust
setting must be changed to `Always Trust` instead of `Use System Defaults`


## Install Pivotal Build Service

Download the following files from [Pivnet](https://network.pivotal.io/products/build-service/):
1) Duffle executable for you operating system
1) Pivotal Build Service Bundle

    Create a credentials file that maps the location where the credentials can be found.
    This file will be used by `duffle` during the installation
    
    A template for the file is next:
    ```yaml
    name: build-service-credentials
    credentials:
      - name: kube_config
        source:
          path: "<path to kubeconfig on local machine>"
        destination:
          path: "/root/.kube/config"
      - name: ca_cert
        source:
          path: "<path to CA certificate for registry access>"
        destination:
          path: "/cnab/app/cert/ca.crt"
    ```
    
    This file should be created in `/tmp/credentials.yml` this location can be changed but
    the next command must be updated accordingly
    
1) Import the images bundle

    This step will extract the bundle
    ```bash
    duffle import /tmp/build-service-${version}.tgz -d /tmp/build-service/
    ```
1) Copy the images from the extracted bundle into an internal Image Registry

    Login to the Image Registry where the images will be stored
    ```bash
    docker login <SOME_IMAGE_REGISTRY>
    ```
    **Note** The only caveat at this point is that the images need to be accessible without the need to login to
    the image registry.
    
    Push the images to the Image Registry
    ```bash
    duffle relocate -f /tmp/build-service/*/bundle.json -m /tmp/relocated.json -p <SOME_IMAGE_REGISTRY>
    ```

1) Install Pivotal Build Service
    
    ```bash
    duffle install <my-build-service-installation-name> -c /tmp/credentials.yml  \
        --set domain=<BUILD_SERVICE_DOMAIN> \
        --set kubernetes_env=<PKS_CLUSTER_NAME> \
        --set ingress_ip=<BUILD_SERVICE_IP> \
        --set docker_registry=<DOCKER_REGISTRY> \
        --set registry_username=<REGISTRY_USERNAME> \
        --set registry_password=<REGISTRY_PASSWORD> \
        --set uaa_url=<UAA_URL> \
        -f /tmp/build-service/*/bundle.json
        -m /tmp/relocated.json
    ```
    
    Variables information:
    
    - `my-build-service-installation-name` this is the unique name for the installation. 
    This name can be used after for upgrading Pivotal Build Service in the cluster `kubectl` is pointing at
    - `BUILD_SERVICE_DOMAIN` is the domain name that will be used to target Pivotal Build Service.
    This domain should have been configured as the domain for the Ingress controller.
    - `PKS_CLUSTER_NAME` Name of the PKS cluster where Pivotal Build Service will be installed
    - `BUILD_SERVICE_IP` IP address that the `BUILD_SERVICE_DOMAIN` should resolve to
    - `DOCKER_REGISTRY` Image Registry used in the previous step to push images to
    - `REGISTRY_USERNAME` Username to access the registry
    - `REGISTRY_PASSWORD` Password to access the registry
    - `UAA_URL` URL to access UAA
    
    **Note** Some images will be pushed again to the image registry because during installation the CA Certificate provided
    will be added to the list of the available CA on these images. To do this, the duffle command must be provided
    with the credentials for the image registry

1) Verify installation

    Download `pb` binary from Pivnet
    
    Target Pivotal Build Service
    ```bash
    pb api <PIVOTAL_BUILD_SERVICE_DOMAIN>
    ```
    
    A user should be created at this point, please follow the instructions in [here](#users-create)
    
    After creating a UAA user the next step should successfully log you in to Pivotal Build Service
    ```bash
    pb login
    ```

## <a id='faq'></a> FAQ

### <a id='users-create'></a> How do I create users to use with Pivotal Build Service?

#### Prerequisites
- Ruby

#### Create the UAA users

The users of Pivotal Build Service are configured on a UAA.
To create these users we recommend using `uaac` tool.

Follow the steps in <a url="#uaa-client-creation">Create the UAA client</a> to
install `uaac` client tool

Target the UAA that will be used to authenticate the Build Service Users

```bash
uaac target <UAA_URL> --skip-ssl-validation
```

Command to create a single user:

```bash
uaac user add <username> -p <password> --emails <email>
```
