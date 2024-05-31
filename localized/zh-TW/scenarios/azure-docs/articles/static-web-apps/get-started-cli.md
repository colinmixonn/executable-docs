---
title: 快速入門：使用 CLI 使用 Azure Static Web Apps 建置您的第一個靜態網站
description: 瞭解如何使用 Azure CLI 將靜態網站部署至 Azure Static Web Apps。
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: quickstart
ms.date: 03/21/2024
ms.author: cshoe
ms.custom: 'mode-api, devx-track-azurecli, innovation-engine, linux-related-content'
ms.devlang: azurecli
---

# 快速入門：使用 Azure CLI 建置您的第一個靜態網站

[![部署至 Azure](https://aka.ms/deploytoazurebutton)](https://go.microsoft.com/fwlink/?linkid=2262845)

Azure Static Web Apps 會透過從程式代碼存放庫建置應用程式，將網站發佈至生產環境。

在本快速入門中，您會使用 Azure CLI 將 Web 應用程式部署至 Azure 靜態 Web 應用程式。

## 必要條件

- [GitHub](https://github.com) 帳戶。
- [Azure](https://portal.azure.com) 帳戶。
  - 如果您沒有 Azure 訂用帳戶，您可以 [建立免費試用帳戶](https://azure.microsoft.com/free)。
- [已安裝 Azure CLI](/cli/azure/install-azure-cli) （2.29.0 版或更高版本）。
- [Git 設定](https://www.git-scm.com/downloads)。 

## 定義環境變數

本快速入門的第一個步驟是定義環境變數。

```bash
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="myStaticWebAppResourceGroup$RANDOM_ID"
export REGION=EastUS2
export MY_STATIC_WEB_APP_NAME="myStaticWebApp$RANDOM_ID"
```

## 建立存放庫 （選擇性）

（選擇性）本文使用 GitHub 範本存放庫作為另一種方式，讓您輕鬆開始使用。 此範本包含要部署至 Azure Static Web Apps 的入門應用程式。

1. 流覽至下列位置以建立新的存放庫： https://github.com/staticwebdev/vanilla-basic/generate。
2. 將您的存放函式庫 `my-first-static-web-app`命名為 。

> [!NOTE]
> Azure Static Web Apps 至少需要一個 HTML 檔案才能建立 Web 應用程式。 您在此步驟中建立的存放庫包含單 `index.html` 一檔案。

3. 選取 [建立存放庫]****。

## 部署靜態 Web 應用程式

從 Azure CLI 將應用程式部署為靜態 Web 應用程式。

1. 建立資源群組。

```bash
az group create \
  --name $MY_RESOURCE_GROUP_NAME \
  --location $REGION
```

結果：
<!-- expected_similarity=0.3 -->
```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-swa-group",
  "location": "eastus2",
  "managedBy": null,
  "name": "my-swa-group",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

2. 從您的存放庫部署新的靜態 Web 應用程式。

```bash
az staticwebapp create \
    --name $MY_STATIC_WEB_APP_NAME \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --location $REGION 
```

部署靜態應用程式有兩個層面。 第一個作業會建立構成您應用程式的基礎 Azure 資源。 第二個是建置和發佈應用程式的工作流程。

您必須先完成執行部署組建，才能移至新的靜態月臺。

3. 返回主控台視窗，然後執行下列命令以列出網站的URL。

```bash
export MY_STATIC_WEB_APP_URL=$(az staticwebapp show --name  $MY_STATIC_WEB_APP_NAME --resource-group $MY_RESOURCE_GROUP_NAME --query "defaultHostname" -o tsv)
```

```bash
runtime="1 minute";
endtime=$(date -ud "$runtime" +%s);
while [[ $(date -u +%s) -le $endtime ]]; do
    if curl -I -s $MY_STATIC_WEB_APP_URL > /dev/null ; then 
        curl -L -s $MY_STATIC_WEB_APP_URL 2> /dev/null | head -n 9
        break
    else 
        sleep 10
    fi;
done
```

結果：
<!-- expected_similarity=0.3 -->
```HTML
<!DOCTYPE html>
<html lang=en>
<head>
<meta charset=utf-8 />
<meta name=viewport content="width=device-width, initial-scale=1.0" />
<meta http-equiv=X-UA-Compatible content="IE=edge" />
<title>Azure Static Web Apps - Welcome</title>
<link rel="shortcut icon" href=https://appservice.azureedge.net/images/static-apps/v3/favicon.svg type=image/x-icon />
<link rel=stylesheet href=https://ajax.aspnetcdn.com/ajax/bootstrap/4.1.1/css/bootstrap.min.css crossorigin=anonymous />
```

```bash
echo "You can now visit your web server at https://$MY_STATIC_WEB_APP_URL"
```

## 使用 GitHub 範本

您已使用 Azure CLI 成功將靜態 Web 應用程式部署至 Azure Static Web Apps。 既然您已基本瞭解如何部署靜態 Web 應用程式，您可以探索 Azure Static Web Apps 的更進階特性和功能。

如果您想要使用 GitHub 樣本存放庫，請遵循下列步驟：

移至 並 https://github.com/login/device 輸入您從 GitHub 取得的程式代碼，以啟動並擷取您的 GitHub 個人存取令牌。

1. 移至 https://github.com/login/device。
2. 輸入主控台訊息顯示的使用者代碼。
3. 選取 `Continue`。
4. 選取 `Authorize AzureAppServiceCLI`。

### 透過 Git 檢視網站

1. 當您在執行文稿時取得存放庫 URL 時，請複製存放庫 URL 並將它貼到瀏覽器中。
2. 選取 [`Actions`] 索引標籤。

   此時，Azure 會建立資源以支援靜態 Web 應用程式。 等候執行中工作流程旁的圖示變成綠色背景的複選標記。 此作業可能需要幾分鐘的時間才能執行。

3. 成功圖示出現之後，工作流程就會完成，而且您可以返回控制台視窗。
4. 執行下列命令來查詢網站的 URL。
```bash
   az staticwebapp show \
     --name $MY_STATIC_WEB_APP_NAME \
     --query "defaultHostname"
```
5. 將 URL 複製到瀏覽器以移至您的網站。

## 清除資源 （選擇性）

如果您不打算繼續使用此應用程式，請使用 az group delete[ 命令刪除資源群組和靜態 Web 應用程式](/cli/azure/group#az-group-delete)。

## 下一步

> [!div class="nextstepaction"]
> [新增 API](add-api.md)
