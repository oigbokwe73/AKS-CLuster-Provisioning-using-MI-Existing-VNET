# AKS-CLuster-Provisioning-using-MI-Existing-VNET


To **create an AKS cluster** using a **User Assigned Managed Identity (UAMI)** that has **Contributor scope on a resource group** and assign it to an **existing VNet subnet**, follow these steps using **Azure CLI**.

---

### ✅ **Prerequisites**

1. Azure CLI installed (`az version >= 2.20.0`)
2. Logged in (`az login`)
3. AKS CLI extension installed:

   ```bash
   az extension add --name aks-preview
   az extension update --name aks-preview
   ```

---

### 🔁 **Step-by-Step with Azure CLI**

---

### **Step 1: Set variables**

```bash
# General settings
$LOCATION="eastus"
$RG_NAME="myResourceGroup"
$AKS_CLUSTER_NAME="myAKSCluster"

# Existing VNet and Subnet
$VNET_NAME="myVnet"
$SUBNET_NAME="mySubnet"

# UAMI
$UAMI_NAME="myAksIdentity"
```

---

### **Step 2: Get subnet ID**

```bash
SUBNET_ID=$(az network vnet subnet show \
  --resource-group $RG_NAME \
  --vnet-name $VNET_NAME \
  --name $SUBNET_NAME \
  --query id -o tsv)
```

---

### **Step 3: Create User Assigned Managed Identity**

> ⚠️ Skip this if identity already exists.

```bash
az identity create \
  --name $UAMI_NAME \
  --resource-group $RG_NAME \
  --location $LOCATION
```

---

### **Step 4: Get the UAMI resource ID**

```bash
UAMI_ID=$(az identity show \
  --name $UAMI_NAME \
  --resource-group $RG_NAME \
  --query id -o tsv)
```

---

### **Step 5: Assign Contributor role to the UAMI**

> Contributor access **on the AKS resource group**

```bash
az role assignment create \
  --assignee $UAMI_ID \
  --role "Contributor" \
  --scope $(az group show --name $RG_NAME --query id -o tsv)
```

---

### **Step 6: Enable subnet permissions for the UAMI**

> Required for AKS to use the subnet.

```bash
# Assign "Network Contributor" on the subnet
az role assignment create \
  --assignee $UAMI_ID \
  --role "Network Contributor" \
  --scope $SUBNET_ID
```

---

### **Step 7: Create the AKS Cluster**

```bash
az aks create \
  --resource-group $RG_NAME \
  --name $AKS_CLUSTER_NAME \
  --location $LOCATION \
  --enable-managed-identity \
  --assign-identity $UAMI_ID \
  --network-plugin azure \
  --vnet-subnet-id $SUBNET_ID \
  --node-count 2 \
  --generate-ssh-keys
```

---

### ✅ **Result**

You’ll now have an AKS cluster:

* Using a **User Assigned Managed Identity**
* Deployed in an **existing VNet Subnet**
* UAMI has Contributor access to the resource group

