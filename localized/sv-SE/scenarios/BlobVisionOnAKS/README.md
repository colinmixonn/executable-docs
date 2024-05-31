# Env vars
export MY_RESOURCE_GROUP_NAME=dasha-vision-test export MY_LOCATION=westus export MY_STORAGE_ACCOUNT_NAME=dashastoragevision export MY_DATABASE_SERVER_NAME=dasha-server-vision2 export MY_DATABASE_NAME=demo export MY_DATABASE_USERNAME=postgres export MY_DATABASE_PASSWORD=Sup3rS3cr3t!
export MY_COMPUTER_VISION_NAME=dasha-vision-test export MY_CONTAINER_APP_NAME=dasha-container-vision export MY_CONTAINER_APP_ENV_NAME=dasha-environment-vision export AKS_SUBNET_NAME=AKSSubnet export POSTGRES_SUBNET_NAME=PostgreSQLSubnet export MY_VNET_NAME=vision-vnet export MY_CONTAINER_REGISTRY=dashavisionacr export MY_CLUSTER_NAME=vision-cluster export DIR="$(dirname "$0")"

##################################################  Skapa AKS-kluster + containerregister (12 m 52,863s)  ##################################################
# 1.4264s P: 2 
az group create --name $MY_RESOURCE_GROUP_NAME --location $MY_LOCATION 

# 17.7362s P:7
az network vnet create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --location $MY_LOCATION \
  --name $MY_VNET_NAME \
  --address-prefix 10.0.0.0/16 \
  --subnet-name $AKS_SUBNET_NAME \
  --subnet-prefix 10.0.1.0/24 \
  --undernät "[{'name':'$POSTGRES_SUBNET_NAME', 'addressPrefix':'10.0.2.0/24'}]"

# 2.3869s P:3
subnetId=$(az network vnet subnet show --name $AKS_SUBNET_NAME --resource-group $MY_RESOURCE_GROUP_NAME --vnet-name $MY_VNET_NAME --query "id" -o tsv)  

# Lägga till Microsoft.Storage-slutpunkten i undernätet så att den kan komma åt postgres senare 
# 13.3114s P:4
az network vnet subnet update \
  --name $AKS_SUBNET_NAME \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --vnet-name $MY_VNET_NAME \
  --service-endpoints Microsoft.Storage 

# Skapa ACR som ska innehålla programmet 
# 37.7627s P:3
az acr create -n $MY_CONTAINER_REGISTRY -g $MY_RESOURCE_GROUP_NAME --sku basic 

#4.7910s P:1
az acr login --name $MY_CONTAINER_REGISTRY   

# Namnet på avbildningen som ska skapas och distribueras till ACR
export IMAGE=$MY_CONTAINER_REGISTRY.azurecr.io/vision-demo:v1

# Skapa AKS i AKS-undernät med anslutning till ACR 
# 224.9959s P: 8
az aks create -n $MY_CLUSTER_NAME -g $MY_RESOURCE_GROUP_NAME --generate-ssh-keys --attach-acr $MY_CONTAINER_REGISTRY --vnet-subnet-id $subnetId --network-plugin azure --service-cidr 10.1.0.0/16 --dns-service-ip 10.1.0.10  

# 2.3341s 2
az aks get-credentials -g $MY_RESOURCE_GROUP_NAME -n $MY_CLUSTER_NAME 

# Skapa avbildningen. TODO: Du kan behöva uppdatera detta för att ta bort "buildx" eftersom det är för M1 Mac är bara att jag utvecklar på
# 133.4897s P:2
docker buildx build --platform=linux/amd64 -t $IMAGE . 

# Push-överföra avbildning till ACR 
# 53.5264s P:1
docker push-$IMAGE 

##################################################  Skala bloblagring  ##################################################
# Lagringskonto 
# 27.3420s P:7
az storage account create --name $MY_STORAGE_ACCOUNT_NAME --resource-group $MY_RESOURCE_GROUP_NAME --location $MY_LOCATION --sku Standard_LRS --vnet-name $MY_VNET_NAME --subnet $AKS_SUBNET_NAME --allow-blob-public-access true

# Lagringskontonyckel 
# 1.9883s P:2
export STORAGE_ACCOUNT_KEY=$(az storage account keys list --account-name $MY_STORAGE_ACCOUNT_NAME --resource-group $MY_RESOURCE_GROUP_NAME --query "[0].value" --output tsv) 

# Lagringscontainer 
# 1.5613s P:4
az storage container create --name images --account-name $MY_STORAGE_ACCOUNT_NAME --account-key $STORAGE_ACCOUNT_KEY --public-access blob 

# # 12.4040s P:7
# az storage cors add \
#   --services b \
#   --methods DELETE GET HEAD MERGE OPTIONS POST PUT \
#   --origins $CLUSTER_INGRESS_URL \
#   --allowed-headers '*' \
#   --max-age 3600 \
#   --account-name $MY_STORAGE_ACCOUNT_NAME \
#   --account-key $STORAGE_ACCOUNT_KEY 


##################################################  Skapa PSQL-databas  ##################################################
# PSQL-databas som skapats i Vnet i Postgres-undernät 
# 330.8194s P:13
az postgres flexible-server create \
  --name $MY_DATABASE_SERVER_NAME \
  --database-name $MY_DATABASE_NAME \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --location $MY_LOCATION \
  --tier Burstable \
  --sku-name Standard_B1ms \
  --storage-size 32 \
  --version 15 \
  --admin-user $MY_DATABASE_USERNAME \
  --admin-password $MY_DATABASE_PASSWORD \
  --vnet $MY_VNET_NAME \
  --subnet $POSTGRES_SUBNET_NAME \
  --Ja 

# PSQL-databas anslutningssträng
export DATABASE_URL="postgres://$MY_DATABASE_USERNAME:$MY_DATABASE_PASSWORD@$MY_DATABASE_SERVER_NAME.postgres.database.azure.com/$MY_DATABASE_NAME" 


##################################################  Skapa visuellt innehåll  ##################################################
# Kräver ett manuellt steg i portalen idag
# Få det här felet om du inte gör det: 
# (ResourceKindRequireAcceptTerms) Den här prenumerationen kan inte skapa ComputerVision förrän du godkänner ansvarsfulla AI-villkor för den här resursen. Du kan godkänna ansvarsfulla AI-villkor genom att skapa en resurs via Azure-portalen och sedan försöka igen. Mer information finns i https://go.microsoft.com/fwlink/?linkid=2164911
# Kod: ResourceKindRequireAcceptTerms
# Meddelande: Den här prenumerationen kan inte skapa ComputerVision förrän du godkänner ansvarsfulla AI-villkor för den här resursen. Du kan godkänna ansvarsfulla AI-villkor genom att skapa en resurs via Azure-portalen och sedan försöka igen. Mer information finns i https://go.microsoft.com/fwlink/?linkid=2164911

# Visuellt innehåll
# 1.8069s P:6
az cognitiveservices account create \
  --name $MY_COMPUTER_VISION_NAME \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --kind ComputerVision \
  --sku S1 \
  --location $MY_LOCATION \
  --Ja   

# Slutpunkt för visuellt innehåll
# 1.2103s P:2
export COMPUTER_VISION_ENDPOINT=$(az cognitiveservices account show --name $MY_COMPUTER_VISION_NAME --resource-group $MY_RESOURCE_GROUP_NAME --query "properties.endpoint" --output tsv) 

# Nyckel för visuellt innehåll
# 1.3998s P:2
exportera COMPUTER_VISION_KEY=$(az cognitiveservices account keys list --name $MY_COMPUTER_VISION_NAME --resource-group $MY_RESOURCE_GROUP_NAME --query "key1" --output tsv)

##################################################  Installera ingressstyrenhet och distribuera program (1m 26.3481s)  ##################################################

# Installera Nginx ingress controller TODO: Kanske vill uppdatera till App Gateway 
# 0.2217s P:1
helm-lagringsplats, lägg till ingress-nginx https://kubernetes.github.io/ingress-nginx 
# 21.0756s P:3
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --create-namespace \
  --namespace ingress-basic \
  --set controller.service.annotations." service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz 

# Ersätta miljövariabler i distributionsmallen med variabler i skript och skapa en ny distributionsmall som ska distribueras på AKS
sed -e "s|<IMAGE_NAME>|${IMAGE}|g" \
  -e "s|<DATABASE_URL>|${DATABASE_URL}|g" \
  -e "s|<COMPUTER_VISION_KEY>|${COMPUTER_VISION_KEY}|g" \
  -e "s|<COMPUTER_VISION_ENDPOINT>|${COMPUTER_VISION_ENDPOINT}|g" \
  -e "s|<MY_STORAGE_ACCOUNT_NAME>|${MY_STORAGE_ACCOUNT_NAME}|g" \
  -e "s|<STORAGE_ACCOUNT_KEY>|${STORAGE_ACCOUNT_KEY}|g" deployment/scripts/deployment-template.yaml > ./deployment.yaml

# 1.9233s + 5s för ingress
kubectl apply -f ./deployment.yaml

# Väntar på att ingresskontrollanten ska distribueras. Fortsätter att kontrollera tills den har distribuerats
medan sant; do aks_cluster_ip=$(kubectl get ingress -o=jsonpath='{.status.loadBalancer.ingress[0].ip}') om [[ -n "$aks_cluster_ip" ]]; echo sedan "AKS Ingress IP Address is: $aks_cluster_ip" break else echo "Waiting for AKS Ingress IP Address to be assigned..." sömn 150s fi gjort

# Problem: Dum att du måste ange Http för ursprunget. Bör bara fungera med IP-adress
export CLUSTER_INGRESS_URL="http://$aks_cluster_ip" 

##################################################  Lägga till CORS i lagringskontot  ##################################################
# Lägga till containerslutpunkt till tillåtet CORS-ursprung för lagringskonto
# 12.4040s P:7
az storage cors add \
  --services b \
  --methods DELETE GET HEAD MERGE OPTIONS POST PUT \
  --origins $CLUSTER_INGRESS_URL \
  --allowed-headers '*' \
  --max-age 3600 \
  --account-name $MY_STORAGE_ACCOUNT_NAME \
  --account-key $STORAGE_ACCOUNT_KEY 


echo "---------- Deployment Complete ----------" echo "AKS Ingress IP Address: $aks_cluster_ip" echo "För att komma åt AKS-klustret använder du följande kommando:" eko "az aks get-credentials -g $MY_RESOURCE_GROUP_NAME -n aks-terraform-cluster" eko ""