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
- An Ingress Controller has been installed on the cluster where Pivotal Build Service is going to be installed. The Pivotal Build Service expects an Ingress Controller and an Ingress Service to be running to configure it's own ingress. A example of an Ingress Controller Deployment can be found [here](https://community.pivotal.io/s/article/how-to-set-up-an-ingress-controller-for-a-pks-cluster)
- Persistent Volumes have been configured on the cluster where Pivotal Build Service will be installed as it relies on Persistent Volume Claims to cache build artifacts to speed up subsequent builds. The cache size per image has been configured to 2GiBs


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

These commands only need to be run once for the full installation.

## <a id='uaa-client-creation'></a> Install UAA Pivotal Build Service client

### Prerequisites
- Ruby

### Install the UAA client

The users of Pivotal Build Service are configured on a UAA.
In order to interact with UAA, Pivotal Build Service must have a UAA client
configured. To configure this client we recommend using `uaac` tool.

1) Install `uaac` tool on your machine, run the following command:

  ```bash
  gem install cf-uaac
  ```

  **Note:** if not using `rbenv` or `rvm` you may need to execute `sudo gem install cf-uaac`

2) Target the UAA that will be used to authenticate the Build Service Users

  ```bash
  uaac target <UAA_URL>
  ```

  **Note:** When using a self-signed certificate, you must use the `--skip-ssl-validation` flag in conjunction with `uaac` 

3) Login as the user management admin user

  ```bash
  uaac token client get admin -s <user-management-admin-user>
  ```
  
  **Note:** This password can be found in the UAA credentials section from OpsMgr.

4) Install the UAA Client

  ```bash
  uaac client add pivotal_build_service_cli --scope="openid,scim.read" --secret="" --authorized_grant_types="password,refresh_token,client_credentials" --authorities="scim.read" --access_token_validity 600 --refresh_token_validity 21600
  ```
  
  **Note:** This command must be executed as is. The secret in this case **must** be an empty string.

## Configure TLS certificates for Pivotal Build Service

You need to get or create TLS certificates for the Pivotal Build Service Domain that will be used in the [Install Pivotal Build Service step](#install-pivotal-build-service). These certificates may be self signed.

**Note:** When using a self-signed certificate, you must use the `--skip-ssl-validation` flag in conjunction with `pb`

When you have the `.crt` and `.key` files, note the paths to these files.
Pass the paths to these files into the `credentials.yml` described in the following steps.

After installation the TLS certificates may be removed.

**NOTES (MacOS Only):** When using the `pb` cli, the CA certificate must be added to the keychain and the trust settings must be changed from `Use System Defaults` to `Always Trust`.


## Install Pivotal Build Service

1) Download the Duffle executable for you operating system from [Pivnet](https://network.pivotal.io/products/build-service/)

1) Download the Pivotal Build Service Bundle from [Pivnet](https://network.pivotal.io/products/build-service/)

1) Create a credentials file that maps the location where the credentials can be found.
   This file will be used by `duffle` during the installation

   Here is a template for the credentials file:
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
     - name: tls_cert
       source:
         path: "<path to TLS certificate>"
       destination:
         path: "/cnab/app/cert/tls.crt"
     - name: tls_key
       source:
         path: "<path to TLS private key>"
       destination:
         path: "/cnab/app/cert/tls.key"
   ```

    Credentials information:

    - `kube_config`: The configuration file required during the installation to interact with the target cluster.

    - `ca_cert`: The CA certificate required by Pivotal Build Service to interact with internally deployed registries. This is the CA certificate that was used while deploying the registry.

    - `tls_cert`: The certificate required for authenticated communication between the `pb` cli and the deployed Build Service. The CA for this certificate must be trusted by the workstation communicating with Pivotal Build Service.

    - `tls_key`: The private key corresponding to the above certificate.

   This file should be created in `/tmp/credentials.yml` this location can be changed but
   the `duffle install` command must be updated accordingly.

   **Note:** In the credentials file all the local paths need to be absolute.

1) Create a parameters file that specifies the parameters for the installation.
   This file will be used by `duffle` during the installation. This file can optionally be replaced by using the `--set` flag to set parameters explicitly as covered later.

    Here is a template for the `parameters.json` file
   ```json
    {
      "domain": "<BUILD SERVICE DOMAIN>",
      "kubernetes_env": "<CLUSTER NAME>",
      "docker_registry": "<DOCKER_REGISTRY>",
      "registry_username": "<REGISTRY_USERNAME>",
      "registry_password": "<REGISTRY_PASSWORD>",
      "uaa_url": "<UAA_URL>",
      "ingress_annotations": {
        "kubernetes.io/ingress.example-annotation-key": "example-annotation-value",
        "kubernetes.io/ingress.example-annotation-key-other": "additional-annotation-value"
      }
    }
   ```
   This file should be created in `/tmp/parameters.json`. This location can be changed but
   the `duffle install` command must be updated accordingly.

   Parameters information:

   - `build_service_domain` is the domain name that will be used to target Pivotal Build Service.
   This domain should have been configured as the domain for the Ingress controller.
   - `pks_cluster_name` Name of the PKS cluster where Pivotal Build Service will be installed
   - `docker_registry` Domain of the Image Registry used in the previous step to push images to

     **Note:** if using dockerhub the domain should be `index.docker.io`
     The registry should not include subpaths in the registry. `gcr.io`, `acr.io` are examples of valid fields for the registry.
   - `registry_username` Username to access the registry (`gcr.io` expects `_json_key` as the username when using the `JSON Key File` auth)
   - `registry_password` Password to access the registry
   - `uaa_url` URL to access UAA

   Additional optional properties:
   - `disable_builder_polling` this will prevent the build service from polling builder images for buildpack updates
   This option requires you to set up a [Builder Webhook](https://github.com/pivotal-cf/docs-build-service/blob/master/webhooks.md).
   This is a boolean value that defaults to `false`.
   - `ingress_annotations` this will set ingress annotations (see the "Optional: Setting custom Ingress controller annotations" step below). This needs to be supplied inside the `parameters.json` file and not via the `--set` flag.
   - `replica_count` this will define the number of build service instances running. It defaults to `1`.
   - `no_gateway` this will install the Pivotal Builder and [kpack](https://github.com/pivotal/kpack): the Build Service CRDs and controllers. This allows one to interact with Build Service via `kubectl` ONLY. This is a boolean value that defaults to `false`.
   (see the "Optional: kpack Only Installation" step below)

   **Note** Some images will be pushed again to the image registry because during installation the CA Certificate provided
   will be added to the list of the available CA on these images. To do this, the parameters file or duffle install command (in case one would like to avoid using the `parameters.json`) must be provided
   with the credentials for the image registry

    ##### Optional: Setting custom Ingress controller annotations
    To use an ingress controller other than NGINX or to override default annotations
    used by the ingress controller, add the annotations to the `ingress_annotations` section of the parameters JSON file.

    ```json
    {
      "ingress_annotations": {
        "kubernetes.io/ingress.my-annotation-key": "my-annotation-value",
        ...
      }
    }
    ```

    ##### Optional: kpack Only Installation
    This will install the Pivotal builder, Build Service CRDs and their controllers. `kubectl` is the CLI tool required to interact with these components. When installing without the gateway, the following parameters are no longer required:
    - `domain`
    - `uaa_url`
    - `ingress_annotations`

    Additionally, one will not need to provide values to the following credentials in `credentials.yml`:
    - `tls_cert`
    - `tls_key`

    Replace the values of the `source.path` properties for these properties with `""` but do not remove them from the file. Removing them will cause the installation to fail.

1) Relocate the images from the extracted bundle into an internal Image Registry

    Login to the Image Registry where the images will be stored
    ```bash
    docker login <SOME_IMAGE_REGISTRY>
    ```

    Push the images to the Image Registry
    ```bash
    duffle relocate -f /tmp/build-service-${version}.tgz -m /tmp/relocated.json -p <TARGET_IMAGE_REGISTRY>
    ```

1) Install Pivotal Build Service

    ```bash
    duffle install <installation-name> -c /tmp/credentials.yml  \
        -p /tmp/parameters.json \
        -f /tmp/build-service-${version}.tgz \
        -m /tmp/relocated.json
    ```
   `installation-name` this is the unique name for the Build Service installation. 
   This name can be used after installation for upgrading Pivotal Build Service in the cluster `kubectl` is pointing at

    One can avoid creating a `parameters.json` file and set parameter values explicitly during the install. For example:
    ```bash
    duffle install <installation-name> -c /tmp/credentials.yml  \
        --set domain=<BUILD_SERVICE_DOMAIN> \
        --set kubernetes_env=<PKS_CLUSTER_NAME> \
        --set docker_registry=<DOCKER_REGISTRY> \
        --set registry_username="<REGISTRY_USERNAME>" \
        --set registry_password="<REGISTRY_PASSWORD>" \
        --set uaa_url=<UAA_URL> \
        -f /tmp/build-service-${version}.tgz \
        -m /tmp/relocated.json
    ```
    A caveat here being, in case you want to set custom ingress annotations, you will have to create a parameters file for it.
    It is possible to have a combination of parameters that are `--set` and passed via the `parameters.json` file by passing the `-p` flag, as long as the keys do not overlap.
    This can lead to errors during installation.

1) Verify installation (for installations that did not set `no_gateway` to `true`)

    Download the `pb` binary from [Pivnet](https://network.pivotal.io/products/build-service/)

    Target Pivotal Build Service
    ```bash
    pb api set <PIVOTAL_BUILD_SERVICE_DOMAIN>
    ```

    In case you have a UAA that has been signed by a self-signed CA, add the `--skip-ssl-validation` flag at the end of the `pb api` command
    
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
