# Terraform Github Action Demo

## Overview

Demo to show how to use github actions and azure enterprise scale landing zones

## Setup

1. Create storage account to store the terraform state
2. Add github secrets to repository
   1. ARM_ACCESS_KEY
   2. ARM_CLIENT_ID
   3. ARM_CLIENT_SECRET
   4. ARM_MGMT_SUBID
   5. ARM_TENANT_ID
   6. AZ_CLI_CREDENTIALS - Paste the output of the sp create command
   7. TERRAFORM_STATE_SA
   8. TERRAFORM_STATE_RG
   9. TERRAFORM_STATE_CONTAINER
   10. PR_GH_TOKEN 


3. Add github secrets to repository
   1. PR_GH_TOKEN = 

Securing pipelines 
Recommended to use multiple different service principals to socpe access


## Create service principal

Create a service principal with RBAC access over the subscription

```powershell
$Random = Get-Random -Minimum 1000 -Maximum 9999
$SpName = "azp-npd-terrademo" + $Random.ToString() + "-sp01"
az ad sp create-for-rbac --name $SpName --scopes /subscriptions/mySubscriptionID --role Contributor 
```

Running the command will output a JSON structure with will be the value for the github secret AZ_CLI_CREDENTIALS

```json
{
  "appId": "<<CLIENT ID>>",
  "displayName": "azp-npd-terrademo8249-sp01",
  "password": "<<CLIENT SECRET>>",
  "tenant": "<<AAD TENANT ID>>"
}
```

## Terraform Storage Account Setup

```powershell
$RESOURCE_GROUP_NAME="ntc-wus3-terra-rg01"
$STORAGE_ACCOUNT_NAME="ntwus3terrasa01"
$CONTAINER_NAME="tf-state-infa"

az group create --name $RESOURCE_GROUP_NAME --location westus3

az storage account create --resource-group $RESOURCE_GROUP_NAME --name $STORAGE_ACCOUNT_NAME --sku Standard_LRS --encryption-services blob

az storage container create --name $CONTAINER_NAME --account-name $STORAGE_ACCOUNT_NAME

$ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query '[0].value' -o tsv)

## CLI ENV Var ##
#export ARM_ACCESS_KEY=$ACCOUNT_KEY

## Powershell ENV Var ##
$env:ARM_ACCESS_KEY=$ACCOUNT_KEY
```
