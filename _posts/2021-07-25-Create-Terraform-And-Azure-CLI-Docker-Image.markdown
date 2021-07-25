---
layout: post
title:  "Create Terraform And Azure CLI Docker Image"
date:   2021-07-03 21:13:31 +0000
categories:
---

# Create Terraform And Azure CLI Docker Image

## Summary

|#|Activities|Description|
|-|-|-|
|1|Create Docker Instance|The Docker instance will contain all the commands line tools neccessary to execute Terraform script against Azure.|
|2|Deploy Azure CLI|Install the official Azure CLI in the Docker instance.|
|3|Deploy Terraform CLI|Install the official Terraform CLI in the Docker instance.|
|4|Connect Terraform CLI to Azure subscription||
|5|Create Docker Image|Bake the instance into a Docker image for future use.|


### Create Docker Instance

1. Create a Docker instance with Ubuntu as base image.

    ```sh
    docker run -it --name terraform-azure ubuntu /bin/bash
    ```

2. Update the package list and install `curl`.

    ```sh
    apt update -y
    apt install -y curl
    ```

### Deploy Azure CLI

Install Azure CLI Documentation: [https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)
1. Execute Azure CLI installation script

    ```
    # In the Docker instance.
    curl -sL https://aka.ms/InstallAzureCLIDeb |  bash
    ```

2. Verify Azure CLI is deployed successfully

    ```
    az login
    ```

    If successful will be promoted to login via browser to enable the CLI connectivity.

    ```sh
    To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code XXXXXXXXX to authenticate.
    ```

    After confirm on the browser to permit the CLI connectivity the subscription details will be shown on the Docker console.
    ```
    [
      {
        "cloudName": "AzureCloud",
        "homeTenantId": "...",
        "id": "...",
        "isDefault": true,
        "managedByTenants": [],
        "name": "...",
        "state": "Enabled",
        "tenantId": "...",
        "user": {
          "name": "...m",
          "type": "user"
        }
      }
    ]
    ```

    The subscription details can also be shown using the following command `az account show`.

### Deploy Terraform CLI

Install Terraform Documentation: [https://learn.hashicorp.com/tutorials/terraform/install-cli](https://learn.hashicorp.com/tutorials/terraform/install-cli)
1. Install Terraform CLI.

    ```sh
    # In the Docker instance.
    apt-get update && sudo apt-get install -y gnupg software-properties-common curl
    apt-get update &&  apt-get install -y gnupg software-properties-common curl
    curl -fsSL https://apt.releases.hashicorp.com/gpg |  apt-key add -
    apt-get update && apt-get install terraform
    ```

2. Verify Terraform is install successfully.

    ```sh
    terraform -v
    ```

    Example output

    ```
    Terraform v1.0.3
    on linux_amd64
    ```

### Connect Terraform CLI to Azure subscription

Official Documentation: [https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)

Terraform provides a few options to connect its CLI to Azure:
* Authenticating to Azure using the Azure CLI
* Authenticating to Azure using Managed Service Identity
* Authenticating to Azure using a Service Principal and a Client Certificate
* Authenticating to Azure using a Service Principal and a Client Secret

For local development option #1 is recommended. This was already performed in the "Deploy Azure CLI" step.

### Create Docker Image

1. After successfully installing the Azure and Terraform CLIs exit the Docker console.
    ```
    exit
    ```

2. Create the Docker image.

    ```
    docker commit terraform-azure localhost/terraform-azure
    ```
