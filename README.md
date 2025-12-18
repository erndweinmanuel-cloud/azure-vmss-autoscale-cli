# azure-vmss-autoscale-cli
Mini Azure Project – Linux VM Scale Set with CPU-based Autoscaling (Azure CLI)

Mini Azure Project – Linux VM Scale Set with CPU-based Autoscaling (Azure CLI)

This mini-lab demonstrates how to deploy and operate an Azure Virtual Machine Scale Set (VMSS) using Azure CLI only, including CPU-based autoscaling (scale-out & scale-in).
The project is part of my AZ-104 learning path and focuses on compute scalability, availability, and automation, without using the Azure Portal for deployment.

---

## Goals

- Create a Resource Group
- Deploy a Linux VM Scale Set (Uniform) via Azure CLI
- Configure CPU-based autoscaling
- Trigger Scale-Out and Scale-In events
- Observe VMSS behavior in a real scenario

---

## Prerequisites

- Azure Subscription
- Azure CLI or Azure Cloud Shell
- Existing SSH key (~/.ssh/id_rsa.pub)
- Basic understanding of:
- Virtual Machines
- Load Balancers
- Metrics & autoscaling concepts

---

## Step 1 - Create Resource Group
```bash
az group create \
  --name rg-vmss-autoscale \
  --location westeurope \
  --output table
```

---

## Step 2 - Deploy Linux VM Scale Set (Uniform)
```bash
az vmss create \
  --resource-group rg-vmss-autoscale \
  --name vmss-linux-autoscale \
  --image Ubuntu2204 \
  --vm-sku Standard_B2s \
  --instance-count 1 \
  --authentication-type ssh \
  --admin-username "enteryouruserhere" \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --orchestration-mode Uniform \
  --upgrade-policy-mode automatic \
  --output table
```
## Result:
- VMSS created with 1 instance
- Standard Load Balancer, VNet, Subnet and NSG are created automatically

---

## Step 3 - Configure Autoscaling (CPU-based)

Create Autoscale Profile
```bash
az monitor autoscale create \
  --resource-group rg-vmss-autoscale \
  --resource vmss-linux-autoscale \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-vmss-cpu \
  --min-count 1 \
  --max-count 3 \
  --count 1
```

Scale-Out Rule 








