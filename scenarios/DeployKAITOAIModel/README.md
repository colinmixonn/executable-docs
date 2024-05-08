# Deploy an AI Model using AI toolchain operator on AKS

## Define Environment Variables
```bash
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="colinRG$RANDOM_ID"
export REGION="eastus"
export MY_OPENAI_RESOURCE_NAME="colinAI$RANDOM_ID"
export MY_DEPLOYMENT_NAME="colinModel$RANDOM_ID"
export CLUSTER_NAME="colinCluster$RANDOM_ID"
```
## Install the Azure CLI preview extension

```bash
az extension add --name aks-preview --allow-preview "true"
```

## Register and Validate the AI toolchain operator add-on feature flag

```bash
az feature register --namespace "Microsoft.ContainerService" --name "AIToolchainOperatorPreview"
```

Results:
<!-- expected_similarity=0.7 -->
```JSON
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.Features/providers/Microsoft.ContainerService/features/AIToolchainOperatorPreview",
  "name": "Microsoft.ContainerService/AIToolchainOperatorPreview",
  "properties": {
    "state": "Registering"
  },
  "type": "Microsoft.Features/providers/features"
}
```
```bash
az feature show --namespace "Microsoft.ContainerService" --name "AIToolchainOperatorPreview"
```

Results:
<!-- expected_similarity=0.7 -->
```JSON
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.Features/providers/Microsoft.ContainerService/features/AIToolchainOperatorPreview",
  "name": "Microsoft.ContainerService/AIToolchainOperatorPreview",
  "properties": {
    "state": "Registered"
  },
  "type": "Microsoft.Features/providers/features"
}
```

## Create an Azure Resource Group

```bash
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
```

Results:
<!-- expected_similarity=0.7 -->
```JSON
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/colinRGxxxxxx",
  "location": "eastus",
  "managedBy": null,
  "name": "colinRGxxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```
## Create an AKS cluster with the AI toolchain operator add-on 

```bash
az aks create --location $REGION --resource-group $MY_RESOURCE_GROUP_NAME --name $CLUSTER_NAME --enable-oidc-issuer --enable-ai-toolchain-operator
```

## Create an Azure OpenAI Resource

```bash
az cognitiveservices account create --name $MY_OPENAI_RESOURCE_NAME --resource-group $MY_RESOURCE_GROUP_NAME --location $REGION --kind OpenAI --sku s0
```