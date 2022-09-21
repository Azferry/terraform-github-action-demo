# Terraform Github Action Demo

Example use case on how to use github actions and terrafrom on azure to accelerate the adoption of enterprise scale landing zones.

## Microsoft Resources

* [What is an Azure landing zone?](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/)
* [Microsoft Docs - Terraform On Azure](https://learn.microsoft.com/azure/developer/terraform/)
* Azure Terraform providers
  * [azurerm](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
  * [azuread](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs)
  * [azapi](https://registry.terraform.io/providers/Azure/azapi/latest/docs)
  * [azstack](https://registry.terraform.io/providers/hashicorp/azurestack/latest/docs)

## Tools used in demo

### Built in Terraform tools

Built in terraform tools to validate the code

* terraform fmt - linter to check if the code is in the correct format
* terraform validate - linter that checks to see if the code is valid
* terraform plan
* terraform apply

### Terraform Docs

Automatically creates documentation from the terraform code and pushes it back into the repository.

### TFSec Code Scanner

> :warning: **GitHub Advanced Security Needed**: GitHub Advanced Security is needed to publish TFSec results to the security tab in the repo. Advanced Security available for enterprise accounts on GitHub Enterprise Cloud, GitHub Enterprise Server, and GitHub Cloud Public repository.

tfsec is a static code security scanner for Terraform code. It ensures that security issues can be detected before your infrastructure changes take effect.

* [GitHub Advanced Security](https://docs.github.com/en/enterprise-cloud@latest/get-started/learning-about-github/about-github-advanced-security)
* [Setup Code Scanning](https://docs.github.com/en/enterprise-server@3.4/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/setting-up-code-scanning-for-a-repository)
* [TFSec Security Rules](https://aquasecurity.github.io/tfsec/v1.28.0/checks/azure/appservice/account-identity-registered/)
* [TFSec Documentation](https://aquasecurity.github.io/tfsec/v1.28.0/guides/github-actions/github-action/)

## Setup

1. Create storage account to store the terraform state
2. Create the github secrets in the repository for actions

### Set the github secrets

Under the repository settings

| Secret                    | Use           | Value                                                        |
| ------------------------- | ------------- | ------------------------------------------------------------ |
| ARM_CLIENT_ID             | Terraform     | Service principal with access to azure                       |
| ARM_CLIENT_SECRET         | Terraform     | Service principal with access to azure                       |
| ARM_TENANT_ID             | Terraform     | Service principal with access to azure                       |
| ARM_ACCESS_KEY            | Terraform     | Storage account key for the TF remote state                  |
| ARM_SUBSCRIPTION_ID       | Terraform     | Subscription ID for the storage account                      |
| TERRAFORM_STATE_SA        | Terraform     | Storage account name for the tf state file                   |
| TERRAFORM_STATE_RG        | Terraform     | Resource group of the storage account                        |
| TERRAFORM_STATE_CONTAINER | Terraform     | Storage account container the state file is in               |
| PR_GH_TOKEN               | Gh Action Bot | Personal access token for github to comment on pull requests |
| AZ_CLI_CREDENTIALS        | Gh Action Bot | Service principal creds in json format (below)               |

## Best Practices

### Azure environment credentials

Use multiple service principals to scope access down to the level of resources.

### Github Action Templates

Create reusable templates to simplify the management of github actions.

### Block code push to main

Enforce all code pushes to go through the pull request process.

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

## Generate Github Token

Used to add comments on the Pull Request

1. https://github.com/settings/tokens
2. Give full access to the repo

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
