```powershell
# create resource group
$resourceGroup = "RG1"
$location = "japaneast"
az group create -n $resourceGroup -l $location

# create virtual network
$vnetName = "demo-vnet"
$vnetAddressPrefix = "10.1.0.0/16"
az network vnet create -n $vnetName -g $resourceGroup --address-prefix $vnetAddressPrefix

# create virtual network subunet
$subnetName = "delegatingsubnet"
az network vnet subnet create `
    -g $resourceGroup `
    --vnet-name $vnetName `
    -n $subnetName `
    --delegations "Microsoft.Web/serverFarms" `
    --address-prefixes "10.1.1.0/24" `
    --service-endpoints "Microsoft.Web"

# create app service plan
$appServicePlanName = "StandardPlan"
az appservice plan create -n $appServicePlanName -g $resourceGroup --sku S1

# create frontend and backend app service
$frontendAppName = "frontend-999f"
az webapp create -n $frontendAppName -g $resourceGroup -p $appServicePlanName
$backendAppName = "backend-999b"
az webapp create -n $backendAppName -g $resourceGroup -p $appServicePlanName

# add restrict to vnet
$subscriptionId = az account show --query id -o tsv
$subnetId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.Network/virtualNetworks/$vnetName/subnets/$subnetName"
$restrictions =  "{ \""ipSecurityRestrictions\"": [ { \""action\"": \""Allow\"", \""vnetSubnetResourceId\"": " + 
                    "\""$subnetId\"", \""name\"": \""LockdownToVNet\"", \""priority\"": 100, \""tag\"": \""Default\"" } ] }"

az webapp config set -g $resourceGroup -n $backendAppName --generic-configurations $restrictions

```
