# Day 2 - Lab 1: VM Baseline Administration

## Context (Why this lab matters)
- VM provisioning and baseline administration are high-frequency AZ-104 tasks.
- This lab establishes a repeatable VM build and verification pattern.
- You should finish with one validated VM baseline ready for later network/app labs.

## Objective
Deploy and validate a small VM baseline in the shared RG.

## Scenario
You need one standard Linux VM baseline for administration practice.
You provision it with cost-optimized defaults and verify operational readiness.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Virtual machines** -> **Create**.
2. Use `rg-az104`, `North Europe`, and VM name `vm-az104-d2-01`.
3. Configure SSH key authentication, apply settings from **Portal VM Defaults (Cost-Optimized)**, and create the VM.
4. After deployment, open the VM **Overview** and verify status is **Running**.
5. Note down VM name, size, and power state.

### Portal VM Defaults (Cost-Optimized)
- **Security tab**
   - Security type: `Standard`
   - Availability options: `No infrastructure redundancy required`
   - Image: `Ubuntu Server (latest available)`
   - Size: choose the lowest-cost size available in the region
   - Inbound ports: allow `SSH (22)` only
   - User: azureuser
   - Create new SSH key, or use your existing one. (RSA and ED25519 are supported)
- **Disks tab**
   - OS disk type: `Standard SSD`
   - Delete with VM: `Enabled`
- **Networking tab**
   - Virtual network: A new Vnet will be auto created for you, as this is your first VM, otherwise pick an existing VNet for the lab context.
   - Subnet: use default subnet unless a lab-specific subnet is explicitly required
   - Public IP: `Create new`
- **Monitoring tab**
   - Diagnostics / Boot diagnostics: `Disabled`

## Azure CLI Helper
```bash
RG=rg-az104
VM=vm-az104-d2-01
az account show --output table
az group show --name $RG --output table
az vm list --resource-group $RG  --output table
az vm get-instance-view --resource-group $RG --name $VM --output table
```

## Optional Deep-Dive Exercise (VM Operations Proof)
Use this only if you want observable admin-state transitions.

### Create
1. Ensure `vm-az104-d2-01` is running.
2. Make a note of initial power state and public IP.

### Validate
1. Stop/deallocate VM:
   - `az vm deallocate --resource-group rg-az104 --name vm-az104-d2-01`
2. Confirm state changes to deallocated:
   - ` az vm get-instance-view --resource-group $RG --name $VM --output table`
3. Start VM again and confirm running state.
  - ` az vm start --resource-group $RG --name $VM`
4. Note down timestamps and resulting states. (VM -> Activity Log)

### Cleanup
1. Return VM to your intended final state (running or deleted per main lab path).
2. Remove any temporary notes/artifacts created only for this optional check.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day2/scenario01/main.json`

## Prerequisites
- `rg-az104` in `northeurope`
- SSH key pair available (or create one)
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
VM=vm-az104-d2-01
PUBKEY=$(cat ~/.ssh/id_rsa.pub)
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day2/scenario01/main.json --parameters location=$LOC vmName=$VM adminPublicKey="$PUBKEY"
az vm get-instance-view --resource-group $RG --name $VM --query instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus -o tsv
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && VM=vm-az104-d2-01 && PUBKEY=$(cat ~/.ssh/id_rsa.pub) && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day2/scenario01/main.json --parameters location=$LOC vmName=$VM adminPublicKey="$PUBKEY" && az vm get-instance-view --resource-group $RG --name $VM --query instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus -o tsv
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `VM=vm-az104-d2-01`
   - `PUBKEY=$(cat ~/.ssh/id_rsa.pub)`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day2/scenario01/main.json --parameters location=$LOC vmName=$VM adminPublicKey="$PUBKEY"`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day2/scenario01/main.json --parameters location=$LOC vmName=$VM adminPublicKey="$PUBKEY"`
4. Check power state:
   - `az vm get-instance-view --resource-group $RG --name $VM --query instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus -o tsv`

## Validation
- VM is running and reachable via Azure control plane.

## Complete Cleanup
- Delete VM and dependent resources:
  - `az vm delete --resource-group $RG --name $VM --yes`
   - `az network nic delete --resource-group $RG --name nic-az104-d2-01 || true`
   - `az network public-ip delete --resource-group $RG --name pip-az104-d2-01 || true`
   - `az network vnet delete --resource-group $RG --name vnet-az104-d2-01 || true`
   - `az network nsg delete --resource-group $RG --name nsg-az104-d2-01 || true`
