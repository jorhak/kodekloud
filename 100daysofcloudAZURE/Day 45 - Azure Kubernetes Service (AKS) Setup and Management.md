```
The Nautilus DevOps team is tasked with preparing an AKS cluster to deploy a Kubernetes-based application. The team has the following requirements:

Create an AKS cluster named nautilus-aks.
The Kubernetes version must be 1.33.0.
The AKS cluster endpoint access must be private.
Ensure the cluster is created in the Central US region.

Edit the agentpool Node pools (delete all other node pool if exists) and configure the cluster with the following properties:

Node size: D2s v3.
Minimum node count: 1.
Maximum node count: 2.

Disable the Container Insights for now and disable all kind of monitoring as well.

The AKS cluster must be configured with high availability and private endpoint access. Verify that the cluster meets the requirements and is ready for workloads.

Use the Azure Portal URL and login credentials below:

Portal URL https://portal.azure.com
Username kk_lab_user_main@azure.onmicrosoft.com
Password contra

Notes:

Create the resources only in the Central US region.
Ensure that the Kubernetes version is 1.33.0.
```
# Variables de entorno
```
AKS_NAME=xfusion-aks
K8S_VERSION=1.33.0
LOCATION=centralus
MIN_NODE_COUNT=1
MAX_NODE_COUNT=2
NODE_SIZE=Standard_D2s_v3
```
# Obtener Resource Group
```
RG_NAME=$(az group list \
   --query [0].name \
   --output tsv)
```
# Crear VNET y SUBNET para Cluster AKS
```
az network vnet create \
  --resource-group "$RG_NAME" \
  --name "aks-vnet" \
  --address-prefixes 10.0.0.0/8 \
  --subnet-name "aks-subnet" \
  --subnet-prefixes 10.240.0.0/16 \
  --location "$LOCATION"
```
# Obtener ID de Subnet
```
SUBNET_ID=$(az network vnet subnet show --resource-group "$RG_NAME" --vnet-name "aks-vnet" --name "aks-subnet" --query id -o tsv)
```
# Crear AKS
```
az aks create \
    --name $AKS_NAME \
    --resource-group $RG_NAME \
    --location $LOCATION \
    --vnet-subnet-id $SUBNET_ID \
    --max-count $MAX_NODE_COUNT \
    --min-count $MIN_NODE_COUNT \
    --node-count $MAX_NODE_COUNT \
    --enable-private-cluster \
    --kubernetes-version $K8S_VERSION \
    --node-vm-size $NODE_SIZE \
    --enable-cluster-autoscaler \
    --no-ssh-key
```
# Verificar
```
az aks show \
  --resource-group "$RG_NAME" \
  --name "$AKS_NAME" \
  --query "{Nombre:name, Version:kubernetesVersion, EsPrivado:apiServerAccessProfile.enablePrivateCluster, Estado:provisioningState}" \
  -o table
```

```
az aks nodepool list \
  --resource-group "$RG_NAME" \
  --cluster-name "$AKS_NAME" \
  --query "[].{Pool:name, SKU:vmSize, Min:enableAutoScaling, MaxCount:maxCount, MinCount:minCount}" \
  -o table
```

```
az aks show \
  --resource-group "$RG_NAME" \
  --name "$AKS_NAME" \
  --query "addonProfiles.omsagent.enabled"
```