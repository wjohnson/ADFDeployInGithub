# CI/CD Workflow for Azure Data Factory with Github Actions

This repo demonstrates how you would take a git enabled Azure Data Factory and deploy it across multiple environments.

Review the [deployadf.yml](./.github\workflows\deployadf.yml) to see the steps involved for Github Actions.

Review the [Continuous integration and delivery in Azure Data Factory overview (MSFT Docs)](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery) for a helpful read on CI/CD in general for Data Factory.

## Setting up the solution

This solution assumes you've already deployed the following resources:
* Three environments consisting of:
  * Azure Data Factory
  * Azure Storage Account (Data Lake Gen 2)
  * Azure Key Vault
* Added the Data Factory to the Access Policies of the Key Vault
* Added the Data Factory as Storage Blob Contributor to the Storage Accounts.

## Github Secrets Required:

|Secret Name| DEV | QA | PRD |
|-----------|-----|----|-----|
| ADF_RESOURCE_ID | DEV_ADF_RESOURCE_ID | N/A | N/A |
| AZURE_CREDENTIALS | NONPRD_AZURE_CREDENTIALS | NONPRD_AZURE_CREDENTIALS | PRD_AZURE_CREDENTIALS |
| AZURE_SUBSCRIPTION | NONPRD_AZURE_SUBSCRIPTION | NONPRD_AZURE_SUBSCRIPTION | PRD_AZURE_SUBSCRIPTION |
| AZURE_RG | DEV_AZURE_RG | QA_AZURE_RG | PRD_AZURE_RG |
| AZURE_KEY_VAULT | DEV_AZURE_KEY_VAULT | QA_AZURE_KEY_VAULT | PRD_AZURE_KEY_VAULT |

* DEV_ADF_RESOURCE_ID: Should follow the pattern `/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.DataFactory/factories/<dfName>`
* AZURE_CREDENTIALS: The JSON response from the [az ad sp create-for-rbac --name {myApp} --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{MyResourceGroup} --sdk-auth](https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) command.
* AZURE_SUBSCRIPTION: The subscription id for this environment.
* AZURE_RG: The name of the resource group for this environment.
* AZURE_KEY_VAULT: The name of the key vault for this environment.

## Key Vault Secrets Required:

For each environment, it's expected that you have a key vault specific to the Azure Data Factory. This would support both the Data Factory linked services as well as the Github Actions.

* data-lake-service-url: The host URL for the Azure Data Lake Service involved in the Data Factory pipeline.

## Using Parameter Files

You may choose to write non-secret parameter values in the `paramaters.<ENV>.json` in this repo instead of using Github or Key Vault secrets to inject them. In this example, I am overwriting the parameter files by specifying parameters `parameters: ./parameters.QA.json factoryName=${{ secrets.QA_FACTORY_NAME }} AzureDataLakeStorage1_properties_typeProperties_url=${{ steps.getAKVSecret.outputs.data-lake-service-url }} AzureKeyVault1_properties_typeProperties_baseUrl=${{ secrets.QA_AZURE_KEY_VAULT }}`. By removing the `variable=value` from the parameters line, you would accept the values provided in the `parameters.<ENV>.json`. Alternatively, if it's not specified in your parameters file, a default value might be provided automatically (some parameter such as linked service secrets will not have a default value).

## Using  Pre and Post Deployment Scripts

As per the [Sample pre- and post-deployment script(MSFT Docs)](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery-sample-script_), a deployment is an incremental operation and will only update assets mentioned in the deployment.

To remove an asset that is no longer referenced in the deployment, you must run an additional script to analyze what assets are no longer mentioned in your deployment and what assets exist.

In addition, when there are triggers active on your data factory, you must pause the triggers so that a deployment may be made. After the deployment, triggers may be reactivated.

The Microsoft docs sample script provide this functionality but it should be reviewed before your use in production.

## References

* Export ADF ARM Templates with [NPM Package](https://docs.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery-improvements)
* [Github Action for Key Vault](https://docs.microsoft.com/en-us/azure/developer/github/github-key-vault)
* [Github Environments and Required Reviewers](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#required-reviewers)
* [Github Action Setup Node](https://github.com/actions/setup-node)
* [Github Action Upload Artifact](https://github.com/actions/upload-artifact)
* [Github Action Download Artifact](https://github.com/actions/download-artifact)
* [Github Action Deploy ARM Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-github-actions)
