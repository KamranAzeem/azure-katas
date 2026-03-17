# Day 2 - Lab 2: VNet + NSG Segmentation

## Context (Why this lab matters)
- Network segmentation and NSG rule intent are central AZ-104 networking skills.
- This lab practices building a simple app/db setup with least-privilege access.
- You should finish with clear subnet separation and validated NSG behavior.

## Objective
Create segmented networking with explicit CIDR planning, per-subnet NSGs, and two VMs (app/db) using least-privilege inbound rules.

## Scenario
You are setting up a basic two-tier environment (`app` and `db`) in one VNet.
Your task is to enforce subnet-level boundaries and verify only intended traffic is allowed.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Virtual networks** -> **Create** and use `rg-az104`.
2. Create VNet `vnet-az104-d2` with address space `10.0.0.0/8`.
3. Create subnet `snet-app` with prefix `10.0.1.0/24` and subnet `snet-db` with prefix `10.0.2.0/24`.
4. Open **Network security groups** (from main portal search) and create:
   - `nsg-az104-d2-app` (for `snet-app`)
   - `nsg-az104-d2-db` (for `snet-db`)
5. Configure inbound NSG rules for both NSGs:
   - `nsg-az104-d2-app`: allow TCP `22` and TCP `80` from **Internet**.
   - `nsg-az104-d2-db`: allow TCP `22` from **Internet**, allow TCP `3306` from source `10.0.1.0/24` only.
6. Associate `nsg-az104-d2-app` to `snet-app` and `nsg-az104-d2-db` to `snet-db`. (Use main VNET -> Subnet -> Edit Subnet -> Network Security Group )
7. Create two Linux VMs in the same RG/region:
   - `vm-az104-d2-app` in `snet-app`
   - `vm-az104-d2-db` in `snet-db`
    - Apply settings from **Portal VM Defaults (Cost-Optimized)**.
8. Validate subnet/NSG associations and NSG rules in Portal.

### Portal VM Defaults (Cost-Optimized)
- It is not wise to use Spot instance for labs, as spot instance can disappear at any time.
- **Security tab**
   - Security type: `Standard`
   - Availability options: `No infrastructure redundancy required`
   - Image: `Ubuntu Server (latest available)`
   - Size: choose the lowest-cost size available in the region
   - Inbound ports: allow `SSH (22)` only
- **Disks tab**
   - OS disk type: `Standard HDD`
   - Delete with VM: `Enabled`
- **Networking tab**
   - Virtual network: pick an existing VNet for the lab context
   - Subnet: use the lab-target subnet (`snet-app` or `snet-db`)
   - Public IP: `Create new`
- **Monitoring tab**
   - Diagnostics / Boot diagnostics: `Disabled`

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --output table
az network vnet show --resource-group $RG --name vnet-az104-d2  --output table
az network vnet show \
  --resource-group $RG \
  --name vnet-az104-d2 \
  --query "addressSpace.addressPrefixes" \
  --output tsv
az network vnet subnet list --resource-group $RG --vnet-name vnet-az104-d2 --output table
az network nsg rule list --resource-group $RG --nsg-name nsg-az104-d2-app --output table
az network nsg rule list --resource-group $RG --nsg-name nsg-az104-d2-db --output table
az vm list --resource-group $RG --query "[?name=='vm-az104-d2-app' || name=='vm-az104-d2-db'].{Name:name,Location:location,Size:hardwareProfile.vmSize}" --output table
```

## Optional Deep-Dive Exercise (Proof of Segmentation)
Use this only if you want observable traffic proof (not just rule inspection).

### Create
1. SSH into `vm-az104-d2-db` and start a temporary listener on port `3306`:
   - `sudo apt-get update && sudo apt-get -y install netcat-openbsd`
   - `nohup nc -lk -p 3306 >/tmp/nc3306.log 2>&1 &`
2. Get DB private IP:
   - `az vm list-ip-addresses --resource-group rg-az104 --name vm-az104-d2-db --query "[0].virtualMachine.network.privateIpAddresses[0]" -o tsv`

### Validate
1. SSH into `vm-az104-d2-app` and test DB reachability on `3306` to DB private IP:
   - `nc -zv <db-private-ip> 3306`
   - Expected: succeeds (allowed from `10.0.1.0/24`).
2. Temporarily break the rule in `nsg-az104-d2-db` (remove/disable `allow-mysql-from-app`) and re-run the same test.
   - Expected: fails or times out.
3. Restore rule `allow-mysql-from-app` and re-test.
   - Expected: succeeds again.

### Cleanup
1. Stop temporary listener on DB VM:
   - `pkill -f "nc -lk -p 3306" || true`
2. Confirm final NSG state still matches baseline lab validation.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day2/scenario02/main.json`

## Prerequisites
- `rg-az104` in `northeurope`
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
VNET=vnet-az104-d2
APP_SNET=snet-app
DB_SNET=snet-db
APP_SNET_PREFIX=10.0.1.0/24
DB_SNET_PREFIX=10.0.2.0/24
APP_NSG=nsg-az104-d2-app
DB_NSG=nsg-az104-d2-db
APP_VM=vm-az104-d2-app
DB_VM=vm-az104-d2-db
ADMIN_USER=azureuser
PUBKEY=$(cat ~/.ssh/id_rsa.pub)
az account show --output table
az group show --name $RG --output table
az network vnet create --resource-group $RG --location $LOC --name $VNET --address-prefixes 10.0.0.0/8 --subnet-name $APP_SNET --subnet-prefixes $APP_SNET_PREFIX
az network vnet subnet create --resource-group $RG --vnet-name $VNET --name $DB_SNET --address-prefixes $DB_SNET_PREFIX
az network nsg create --resource-group $RG --location $LOC --name $APP_NSG
az network nsg create --resource-group $RG --location $LOC --name $DB_NSG
az network nsg rule create --resource-group $RG --nsg-name $APP_NSG --name allow-ssh-internet --priority 100 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes Internet --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 22
az network nsg rule create --resource-group $RG --nsg-name $APP_NSG --name allow-http-internet --priority 110 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes Internet --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80
az network nsg rule create --resource-group $RG --nsg-name $DB_NSG --name allow-ssh-internet --priority 100 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes Internet --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 22
az network nsg rule create --resource-group $RG --nsg-name $DB_NSG --name allow-mysql-from-app --priority 110 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes $APP_SNET_PREFIX --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 3306
az network vnet subnet update --resource-group $RG --vnet-name $VNET --name $APP_SNET --network-security-group $APP_NSG
az network vnet subnet update --resource-group $RG --vnet-name $VNET --name $DB_SNET --network-security-group $DB_NSG
az vm create --resource-group $RG --location $LOC --name $APP_VM --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $APP_SNET --public-ip-sku Standard --nsg ""
az vm create --resource-group $RG --location $LOC --name $DB_VM --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $DB_SNET --public-ip-sku Standard --nsg ""
az network vnet subnet list --resource-group $RG --vnet-name $VNET --query "[].{Subnet:name,Prefix:addressPrefix,Nsg:id}" --output table
az network nsg rule list --resource-group $RG --nsg-name $APP_NSG --query "[].{Name:name,Source:sourceAddressPrefix,Port:destinationPortRange}" --output table
az network nsg rule list --resource-group $RG --nsg-name $DB_NSG --query "[].{Name:name,Source:sourceAddressPrefix,Port:destinationPortRange}" --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && VNET=vnet-az104-d2 && APP_SNET=snet-app && DB_SNET=snet-db && APP_SNET_PREFIX=10.0.1.0/24 && DB_SNET_PREFIX=10.0.2.0/24 && APP_NSG=nsg-az104-d2-app && DB_NSG=nsg-az104-d2-db && APP_VM=vm-az104-d2-app && DB_VM=vm-az104-d2-db && ADMIN_USER=azureuser && PUBKEY=$(cat ~/.ssh/id_rsa.pub) && az account show --output table && az group create --name $RG --location $LOC --output table && az network vnet create --resource-group $RG --location $LOC --name $VNET --address-prefixes 10.0.0.0/8 --subnet-name $APP_SNET --subnet-prefixes $APP_SNET_PREFIX && az network vnet subnet create --resource-group $RG --vnet-name $VNET --name $DB_SNET --address-prefixes $DB_SNET_PREFIX && az network nsg create --resource-group $RG --location $LOC --name $APP_NSG && az network nsg create --resource-group $RG --location $LOC --name $DB_NSG && az network nsg rule create --resource-group $RG --nsg-name $APP_NSG --name allow-ssh-internet --priority 100 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes Internet --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 22 && az network nsg rule create --resource-group $RG --nsg-name $APP_NSG --name allow-http-internet --priority 110 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes Internet --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80 && az network nsg rule create --resource-group $RG --nsg-name $DB_NSG --name allow-ssh-internet --priority 100 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes Internet --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 22 && az network nsg rule create --resource-group $RG --nsg-name $DB_NSG --name allow-mysql-from-app --priority 110 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes $APP_SNET_PREFIX --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 3306 && az network vnet subnet update --resource-group $RG --vnet-name $VNET --name $APP_SNET --network-security-group $APP_NSG && az network vnet subnet update --resource-group $RG --vnet-name $VNET --name $DB_SNET --network-security-group $DB_NSG && az vm create --resource-group $RG --location $LOC --name $APP_VM --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $APP_SNET --public-ip-sku Standard --nsg "" && az vm create --resource-group $RG --location $LOC --name $DB_VM --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $DB_SNET --public-ip-sku Standard --nsg "" && az network vnet subnet list --resource-group $RG --vnet-name $VNET --query "[].{Subnet:name,Prefix:addressPrefix,Nsg:id}" --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `VNET=vnet-az104-d2`
   - `APP_SNET=snet-app` and `DB_SNET=snet-db`
   - `APP_SNET_PREFIX=10.0.1.0/24` and `DB_SNET_PREFIX=10.0.2.0/24`
   - `APP_NSG=nsg-az104-d2-app` and `DB_NSG=nsg-az104-d2-db`
   - `APP_VM=vm-az104-d2-app` and `DB_VM=vm-az104-d2-db`
   - `PUBKEY=$(cat ~/.ssh/id_rsa.pub)`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day2/scenario02/main.json --parameters location=$LOC vnetName=$VNET appSubnetName=$APP_SNET dbSubnetName=$DB_SNET nsgName=$APP_NSG`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day2/scenario02/main.json --parameters location=$LOC vnetName=$VNET appSubnetName=$APP_SNET dbSubnetName=$DB_SNET nsgName=$APP_NSG`
4. In Portal, create second NSG `nsg-az104-d2-db`, add the required app/db rules, attach each NSG to its matching subnet, and create both VMs in their target subnets.
4. Verify resources:
   - `az network vnet show --resource-group $RG --name $VNET --query "{AddressSpace:addressSpace.addressPrefixes}" --output table`
   - `az network vnet subnet list --resource-group $RG --vnet-name $VNET --query "[].{Subnet:name,Prefix:addressPrefix,Nsg:id}" --output table`
   - `az network nsg rule list --resource-group $RG --nsg-name $APP_NSG --output table`
   - `az network nsg rule list --resource-group $RG --nsg-name $DB_NSG --output table`
   - `az vm list --resource-group $RG --query "[?name=='vm-az104-d2-app' || name=='vm-az104-d2-db'].{Name:name,Size:hardwareProfile.vmSize}" --output table`

## Validation
- VNet `vnet-az104-d2` has address space `10.0.0.0/8`.
- `snet-app` uses `10.0.1.0/24` and is associated with `nsg-az104-d2-app`.
- `snet-db` uses `10.0.2.0/24` and is associated with `nsg-az104-d2-db`.
- `nsg-az104-d2-app` allows inbound `22` and `80` from Internet.
- `nsg-az104-d2-db` allows inbound `22` from Internet and `3306` only from `10.0.1.0/24`.
- Both VMs exist in their intended subnets.
- Optional deep-dive path: break/fix test on `3306` confirms segmentation behavior with observable connection results.

## Complete Cleanup
Use Portal to remove resources under the "rg-az104" resource group. Or, use the commands below. (some commands may need a little adjustment)

- `az vm delete --resource-group rg-az104 --name vm-az104-d2-app --yes`
- `az vm delete --resource-group rg-az104 --name vm-az104-d2-db --yes`
- `az network vnet subnet update --resource-group $RG --vnet-name $VNET --name snet-app --network-security-group ""`
- `az network vnet subnet update --resource-group $RG --vnet-name $VNET --name snet-db  --network-security-group ""`
- `az network nsg delete --resource-group rg-az104 --name nsg-az104-d2-app`
- `az network nsg delete --resource-group rg-az104 --name nsg-az104-d2-db`
- `az network vnet delete --resource-group rg-az104 --name vnet-az104-d2`
