---
title: 'Create and deploy Azure OpenAI Service with the Azure CLI'
description: Learn how to get started with Azure OpenAI Service and create your first resource and deploy your first model in the Azure CLI
services: cognitive-services
author: colinmixonn
ms.service: azure-ai-openai
ms.topic:  how-to
ms.date: 05/08/2024
ms.author: colinmixon
ms.custom: innovation-engine, linux-related-content, devx-track-azurecli
ms.devlang: azurecli
---

# Create and Deploy an OpenAI Model using Azure OpenAI Service

## Define Environment Variables
```bash
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="myAIResourceGroup$RANDOM_ID"
export REGION="eastus"
export MY_OPENAI_RESOURCE_NAME="myOAIResource$RANDOM_ID"
export MY_DEPLOYMENT_NAME="myModel$RANDOM_ID"
```
## Create an Azure Resource Group

```azurecli-interactive
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
```

Results:
<!-- expected_similarity=0.7 -->
```JSON
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myAIResourceGroupxxxxxx",
  "location": "eastus",
  "managedBy": null,
  "name": "myAIResourceGroupxxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create an Azure OpenAI Resource

```bash
az cognitiveservices account create --name $MY_OPENAI_RESOURCE_NAME --resource-group $MY_RESOURCE_GROUP_NAME --location $REGION --kind OpenAI --sku s0
```

Results:
<!-- expected_similarity=0.7 -->
```JSON
{
  "etag": "\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myAIResourceGroupxxxxxx/providers/Microsoft.CognitiveServices/accounts/myOAIResourcexxxxxx",
  "identity": null,
  "kind": "OpenAI",
  "location": "eastus",
  "name": "myOAIResourcexxxxxx",
  "properties": {
    "abusePenalty": null,
    "allowedFqdnList": null,
    "apiProperties": null,
    "callRateLimit": {
      "count": null,
      "renewalPeriod": null,
      "rules": [
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.dalle.post",
          "matchPatterns": [
            {
              "method": "POST",
              "path": "dalle/*"
            },
            {
              "method": "POST",
              "path": "openai/images/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.dalle.other",
          "matchPatterns": [
            {
              "method": "*",
              "path": "dalle/*"
            },
            {
              "method": "*",
              "path": "openai/operations/images/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai",
          "matchPatterns": [
            {
              "method": "*",
              "path": "openai/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "default",
          "matchPatterns": [
            {
              "method": "*",
              "path": "*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        }
      ]
    },
    "capabilities": [
      {
        "name": "VirtualNetworks",
        "value": null
      },
      {
        "name": "CustomerManagedKey",
        "value": null
      },
      {
        "name": "MaxFineTuneCount",
        "value": "100"
      },
      {
        "name": "MaxRunningFineTuneCount",
        "value": "1"
      },
      {
        "name": "MaxUserFileCount",
        "value": "50"
      },
      {
        "name": "MaxTrainingFileSize",
        "value": "512000000"
      },
      {
        "name": "MaxUserFileImportDurationInHours",
        "value": "1"
      },
      {
        "name": "MaxFineTuneJobDurationInHours",
        "value": "720"
      },
      {
        "name": "TrustedServices",
        "value": "Microsoft.CognitiveServices,Microsoft.MachineLearningServices,Microsoft.Search"
      }
    ],
    "commitmentPlanAssociations": null,
    "customSubDomainName": null,
    "dateCreated": "xxxx-xx-xxxxx:xx:xx.xxxxxxxx",
    "deletionDate": null,
    "disableLocalAuth": null,
    "dynamicThrottlingEnabled": null,
    "encryption": null,
    "endpoint": "https://eastus.api.cognitive.microsoft.com/",
    "endpoints": {
      "OpenAI Dall-E API": "https://eastus.api.cognitive.microsoft.com/",
      "OpenAI Language Model Instance API": "https://eastus.api.cognitive.microsoft.com/",
      "OpenAI Model Scaleset API": "https://eastus.api.cognitive.microsoft.com/",
      "OpenAI Whisper API": "https://eastus.api.cognitive.microsoft.com/"
    },
    "internalId": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "isMigrated": false,
    "locations": null,
    "migrationToken": null,
    "networkAcls": null,
    "privateEndpointConnections": [],
    "provisioningState": "Succeeded",
    "publicNetworkAccess": "Enabled",
    "quotaLimit": null,
    "restore": null,
    "restrictOutboundNetworkAccess": null,
    "scheduledPurgeDate": null,
    "skuChangeInfo": null,
    "userOwnedStorage": null
  },
  "resourceGroup": "myAIResourceGroupxxxxxx",
  "sku": {
    "capacity": null,
    "family": null,
    "name": "S0",
    "size": null,
    "tier": null
  },
  "systemData": {
    "createdAt": "xxxx-xx-xxxxx:xx:xx.xxxxxxxx",
    "createdBy": "yyyyyyyyyyyyyyyyyyyyyyyy",
    "createdByType": "User",
    "lastModifiedAt": "xxxx-xx-xxxxx:xx:xx.xxxxxx+xx:xx",
    "lastModifiedBy": "yyyyyyyyyyyyyyyyyyyyyyyy",
    "lastModifiedByType": "User"
  },
  "tags": null,
  "type": "Microsoft.CognitiveServices/accounts"
}
```
# Retrieve information about the resource

### Get the endpoint URL

```bash
az cognitiveservices account show --name $MY_OPENAI_RESOURCE_NAME --resource-group  $MY_RESOURCE_GROUP_NAME | jq -r .properties.endpoint
```
Results:
<!-- expected_similarity=0.7 -->
```OUTPUT
https://xxxxxx.api.cognitive.microsoft.com/
```

### Get the primary API key
```bash
az cognitiveservices account keys list --name $MY_OPENAI_RESOURCE_NAME --resource-group  $MY_RESOURCE_GROUP_NAME | jq -r .key1
```

## Deploy a Model

```bash
az cognitiveservices account deployment create --name $MY_OPENAI_RESOURCE_NAME --resource-group $MY_RESOURCE_GROUP_NAME --deployment-name $MY_DEPLOYMENT_NAME --model-name text-embedding-ada-002 --model-version "1" --model-format OpenAI --sku-capacity "1" --sku-name "Standard"
```
Results:
<!-- expected_similarity=0.7 -->
```JSON
{
  "etag": "\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myAIResourceGroupxxxxxx/providers/Microsoft.CognitiveServices/accounts/myOAIResourcexxxxxx/deployments/myModelxxxxxx",
  "name": "myModelxxxxxx",
  "properties": {
    "callRateLimit": null,
    "capabilities": {
      "embeddings": "true",
      "embeddingsMaxInputs": "1"
    },
    "model": {
      "callRateLimit": null,
      "format": "OpenAI",
      "name": "text-embedding-ada-002",
      "source": null,
      "version": "1"
    },
    "provisioningState": "Succeeded",
    "raiPolicyName": null,
    "rateLimits": [
      {
        "count": 1.0,
        "dynamicThrottlingEnabled": null,
        "key": "request",
        "matchPatterns": null,
        "minCount": null,
        "renewalPeriod": 10.0
      },
      {
        "count": 1000.0,
        "dynamicThrottlingEnabled": null,
        "key": "token",
        "matchPatterns": null,
        "minCount": null,
        "renewalPeriod": 60.0
      }
    ],
    "scaleSettings": null,
    "versionUpgradeOption": "OnceNewDefaultVersionAvailable"
  },
  "resourceGroup": "myAIResourceGroupxxxxxx",
  "sku": {
    "capacity": 1,
    "family": null,
    "name": "Standard",
    "size": null,
    "tier": null
  },
  "systemData": {
    "createdAt": "xxxx-xx-xxxxx:xx:xx.xxxxxx+xx:xx",
    "createdBy": "yyyyyyyyyyyyyyyyyyyyyyyy",
    "createdByType": "User",
    "lastModifiedAt": "xxxx-xx-xxxxx:xx:xx.xxxxxx+xx:xx",
    "lastModifiedBy": "yyyyyyyyyyyyyyyyyyyyyyyy",
    "lastModifiedByType": "User"
  },
  "type": "Microsoft.CognitiveServices/accounts/deployments"
}
```
## Delete a Model from your OpenAI Resource

```bash
az cognitiveservices account deployment delete --name $MY_OPENAI_RESOURCE_NAME --resource-group $MY_RESOURCE_GROUP_NAME --deployment-name $MY_DEPLOYMENT_NAME
```

## Delete an OpenAI Resource

```bash
az cognitiveservices account delete --name $MY_OPENAI_RESOURCE_NAME --resource-group $MY_RESOURCE_GROUP_NAME
```