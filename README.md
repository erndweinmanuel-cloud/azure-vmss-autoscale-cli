# azure-vmss-autoscale-cli
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

**Proof - VM Scale Set created**

  <img width="730" height="69" alt="Screenshot 2025-12-18 145450" src="https://github.com/user-attachments/assets/1b08d73e-f8bb-44d1-9c89-17daec4f10fa" />

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

Scale-Out Rule (CPU > 70%)
```bash
az monitor autoscale rule create \
  --resource-group rg-vmss-autoscale \
  --autoscale-name autoscale-vmss-cpu \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 1
```

Scale-In Rule (CPU < 50%)
```bash
az monitor autoscale rule create \
  --resource-group rg-vmss-autoscale \
  --autoscale-name autoscale-vmss-cpu \
  --condition "Percentage CPU < 50 avg 10m" \
  --scale in 1
```

---

## Step 4 - Observe VMSS Scaling Behavior 

List VMMS Instances
```bash
az vmss list-instances \
  --resource-group rg-vmss-autoscale \
  --name vmss-linux-autoscale \
  --query "[].instanceId" \
  --output table
```
## Observed behavior
- Initial state: 1 instance
- Under load: scaled out to 2 and 3 instances
- After load stopped: scaled back down to 2, then 1
- Scale-In takes longer due to averaging windows and cooldown behavior.

**Proof - Scale-Out and Scale-in**

Initial State:

<img width="153" height="87" alt="Screenshot 2025-12-18 145542" src="https://github.com/user-attachments/assets/135f69d6-ac62-437d-9498-b4998cd61d3a" />

Scale-Out (CPU > 70%):

<img width="146" height="128" alt="Screenshot 2025-12-18 151729" src="https://github.com/user-attachments/assets/45fbbb31-272b-4f34-a09c-ce0a06400a37" />

Scale-In (CPU < 50%):

<img width="134" height="98" alt="Screenshot 2025-12-18 151904" src="https://github.com/user-attachments/assets/9ad03cc5-f201-44bb-b217-679147b5d076" />

---

## Section 5 - Cleanup
```bash
az group delete \
  --name rg-vmss-autoscale \
  --yes --no-wait
```

---


## Learnings

- VM Scale Sets provide horizontal scalability for virtual machines
- Autoscale decisions are not instant – scale-in is intentionally slower than scale-out
- Autoscaling is driven by:
- Metric thresholds
- Averaging windows
- Cooldown periods
- VMSS abstracts:
- Load Balancer integration
- Instance lifecycle management
- CLI-only deployments expose real operational details often hidden in the Portal
- VMSS is a foundation service used before higher-level services like App Services or containers

---

## Notes

This project focuses on infrastructure-level scaling, not application deployment

VMSS is commonly used as a building block for:

- High-availability compute
- Backend services
- Autoscaled workloads







