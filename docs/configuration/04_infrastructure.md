# Infrastructure
We will use terraform to configure and deploy the infrastructure used. 

To avoid the use of scripting languages, which will quickly involve one
form or another of a "meta" template language, we use environment variables
to communicate information between tools. 

Using this mechanism with terraform presents us the first challenge.

## TL;DR Intro to Terraform Variables:

### Input Variables
- Variable are defined by convention in `variables.tf`
- [Definitions include](https://www.terraform.io/docs/language/values/variables.html#declaring-an-input-variable) 
    - type constraints, 
    - default values, 
    - description, 
    - validation rules,
    - sensitivity
    example:

- Variables can be specified in one of three ways:
    - [file(s)](https://www.terraform.io/docs/language/values/variables.html#variable-definitions-tfvars-files) with the `.tfvars` extension
    - [command line](https://www.terraform.io/docs/language/values/variables.html#variables-on-the-command-line)
    - [environment variables](https://www.terraform.io/docs/language/values/variables.html#environment-variables)

### Output Values
  - Output values are defined by output blocks. An example
    ```hcl
    output "instance_ip_addr" {
      value       = aws_instance.server.private_ip
      description = "The private IP address of the main server instance."
      sensitive   = false
      depends_on  = [
        # Security group rule must be created before this IP address could
        # actually be used, otherwise the services will be unreachable.
        aws_security_group_rule.local_access,
      ]
    }
    ```
  - Output values can be accessed via the output command
    ```sh
      $ terraform output instance_ip_addr
      "52.4.116.53"
    ```
    We'll use this to set environment variables as needed to continue along our journey.

We have cherry-picked things to fit our need here. If interested, here's the full monty
  - [Input Variables](https://www.terraform.io/docs/language/values/variables.html).
  - [Output Values](https://www.terraform.io/docs/language/values/outputs.html
)

### Globals
We translate our environment variables to a TF_VAR form for terraform

<!-- tabs:start -->

#### **windows**

(globals.bat)

```batch globals.bat
  set ACR_NAME=%APP_NAME%
  set AKS_NAME=%APP_NAME%
  set TF_VAR_region=%REGION%
  set TF_VAR_resource_group=%RESOURCE_GROUP%
  set TF_VAR_aks_name=%AKS_NAME%
  set TF_VAR_acr_name=%ACR_NAME%
```

#### **\*nix**

(globals.sh)

```sh globals.sh
  ACR_NAME=${APP_NAME}
  AKS_NAME==${APP_NAME}
  TF_VAR_region=${REGION}
  TF_VAR_resource_group=${RESOURCE_GROUP}
  TF_VAR_aks_name=${AKS_NAME}
  TF_VAR_acr_name=$ACR_NAME}
```
<!-- tabs:end -->

### Terraform Variables

```hcl terraform/variables.tf
# we use variables.tf to define all variables needed - input and output

variable "region" {
  type = string
}

variable "resource_group" {
  type = string
}

variable "aks_name" {
  type = string
}

variable "acr_name" {
  type = string
}
```

## Azure provider

We'll start by declaring the Azure Provider

```hcl terraform/backstage-azure.tf

terraform {
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "~>2.0"
    }
  }
}

provider "azurerm" {
  features {}
}

```

### Azure Resource Group

```hcl terraform/backstage-azure.tf
resource "azurerm_resource_group" "resource_group" {
  name = var.resource_group
  location = var.region
}
```
