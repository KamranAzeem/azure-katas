# Day 5 - Lab 2: Timed Practical Round 2 (Troubleshooting Sprint)

## Context (Why this lab matters)
- This is Round 2 of Day 5 simulation and should be run under a strict time box.
- AZ-104 operations often require quick diagnosis across Deployments, Activity Log, and network controls.
- You should leave this round with one clearly documented issue and a confirmed fix.
- Important: NSG rule mistakes are often data-plane issues; they may not produce a **Failed** Activity Log event.

## Objective
Complete a timed diagnosis-and-fix cycle for a realistic networking/governance issue in `rg-az104`.

## Scenario
Users report expected HTTP access is blocked after recent NSG changes.
Within the round time limit, identify rule precedence, apply the fix, and note down details of resolved behavior.

## Pre-Lab Setup (Portal, if resources are missing)
Use these steps first if `vnet-az104-d5-troubleshoot`, `snet-app`, or `nsg-az104-d5-troubleshoot` do not exist yet.

> Note: If you deployed `labs/arm/day5/scenario02/main.json`, the known broken rule `deny-http-lab` is already injected. In that case, skip manual injection in step 5.

1. Open Azure Portal -> **Resource groups** -> **rg-az104** and confirm region `North Europe`.
2. Create NSG:
   - Go to **Network security groups** -> **+ Create**
   - Name: `nsg-az104-d5-troubleshoot`
   - Region: `North Europe`
   - Resource group: `rg-az104`
   - Click **Review + create** -> **Create**.
3. Create VNet + subnet:
   - Go to **Virtual networks** -> **+ Create**
   - Name: `vnet-az104-d5-troubleshoot`
   - Region: `North Europe`
   - Resource group: `rg-az104`
   - Address space: `10.250.0.0/16`
   - Subnet name: `snet-app`
   - Subnet address range: `10.250.1.0/24`
   - Click **Review + create** -> **Create**.
4. Associate NSG to subnet:
   - Open **Virtual networks** -> `vnet-az104-d5-troubleshoot` -> **Subnets** -> `snet-app`
   - Set **Network security group** = `nsg-az104-d5-troubleshoot`
   - Click **Save**.
5. Inject a known problem state (Portal):
   - Open **Network security groups** -> `nsg-az104-d5-troubleshoot` -> **Inbound security rules** -> **+ Add**.
   - Create a deny rule:
     - Name: `deny-http-lab`
     - Source: `Any`
     - Source port ranges: `*`
     - Destination: `Any`
     - Service: `Custom`
     - Destination port ranges: `80`
     - Protocol: `TCP`
     - Action: `Deny`
     - Priority: `200`
   - Click **Add**.
6. Keep baseline intentionally simple for troubleshooting:
   - No public IP is required.
   - No VM is required for this lab path.
   - Do not add `allow-http` yet; that rule is the intended fix during troubleshooting.

### Pre-Check (CLI, read-only)
Run these commands to confirm pre-lab setup exists before starting troubleshooting steps:

```bash
RG=rg-az104
az group show --name $RG --output table
az network vnet show --resource-group $RG --name vnet-az104-d5-troubleshoot --query "{Name:name,AddressSpace:addressSpace.addressPrefixes,Location:location}" --output table
az network vnet subnet show --resource-group $RG --vnet-name vnet-az104-d5-troubleshoot --name snet-app --query "{Name:name,Prefix:addressPrefix,Nsg:id}" --output table
az network nsg show --resource-group $RG --name nsg-az104-d5-troubleshoot --query "{Name:name,Location:location,ProvisioningState:provisioningState}" --output table
az network nsg rule list --resource-group $RG --nsg-name nsg-az104-d5-troubleshoot --query "[].{Name:name,Priority:priority,Access:access,Direction:direction,Protocol:protocol,DestinationPort:destinationPortRange}" --output table
```

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Resource groups** -> **rg-az104**.
2. Open **Deployments** and review the latest entries.
   - Check **Status**, **Timestamp**, and failed-operation details.
3. Open **Activity log** for `rg-az104`.
   - Filter by **Status = Failed** first; if empty, switch to **Status = All** and recent time window.
   - For this lab, it is expected that you may only see **Succeeded** control-plane operations (for example NSG rule write/update).
   - Note one relevant operation as details, then pair it with NSG rule-state details.
4. Open **Virtual networks** -> `vnet-az104-d5-troubleshoot` (if present).
   - Go to **Subnets** and confirm subnet `snet-app` exists.
   - Verify effective association to `nsg-az104-d5-troubleshoot` where applicable.
5. Open **Network security groups** -> `nsg-az104-d5-troubleshoot` -> **Inbound security rules**.
   - Review priority/order and identify restrictive or missing rule behavior.
   - Expected pre-lab problem: `deny-http-lab` at priority `200` blocks intended HTTP allow behavior.
6. Apply one fix in Portal:
   - Remove `deny-http-lab` **or** lower its precedence.
   - Add/confirm rule `allow-http` with `TCP`, destination port `80`, action `Allow`, priority `200`.
7. Re-validate:
   - Return to **Activity log** and confirm rule update operation(s) show success.
   - Re-open NSG rules and verify the intended rule now exists with expected priority/protocol/port.
8. Note down troubleshooting outcome (minimum):
   - Problem observed
   - Details used (activity-log operation + NSG rule state)
   - Fix applied (exact NSG/subnet change)
   - Validation result after fix

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az monitor activity-log list --resource-group $RG --max-events 30 --output table
az deployment group list --resource-group $RG --output table
az network nsg rule list --resource-group $RG --nsg-name nsg-az104-d5-troubleshoot --output table
```

## Optional Deep-Dive Exercise (Proof of Impact)
Use this only if you want a concrete traffic test showing the deny/allow effect.

### Create
1. In `rg-az104`, create a Linux VM named `vm-az104-d5-app` in `vnet-az104-d5-troubleshoot` / `snet-app`.
2. During VM creation, allow SSH (`22`) for admin access if prompted.
3. Keep this VM minimal (for example `Standard_B1s`) to reduce cost.
4. Connect by SSH and run:
   - `sudo apt update && sudo apt install -y nginx`
   - `sudo systemctl enable --now nginx`

### Validate
1. Keep `deny-http-lab` active (or remove `allow-http`) and test `http://<vm-public-ip>` from your browser.
   - Expected: timeout/refused/no response on HTTP.
2. Apply the fix (`allow-http`, TCP/80, priority 200, and no conflicting deny with higher precedence).
3. Test `http://<vm-public-ip>` again.
   - Expected: default nginx page loads.
4. Note down both observations as before/after details.

### Cleanup
1. Delete test VM `vm-az104-d5-app` from Portal.
2. Confirm dependent NIC/public IP/disk are deleted (or delete remaining artifacts explicitly).

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day5/scenario02/main.json`

## Prerequisites
- Shared lab resources in `rg-az104`
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
NSG=nsg-az104-d5-troubleshoot
VNET=vnet-az104-d5-troubleshoot
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario02/main.json --parameters location=$LOC nsgName=$NSG vnetName=$VNET subnetName=snet-app
az monitor activity-log list --resource-group $RG --max-events 30 --output table
az network nsg rule list --resource-group $RG --nsg-name $NSG --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && NSG=nsg-az104-d5-troubleshoot && VNET=vnet-az104-d5-troubleshoot && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario02/main.json --parameters location=$LOC nsgName=$NSG vnetName=$VNET subnetName=snet-app && az monitor activity-log list --resource-group $RG --max-events 30 --output table && az network nsg rule list --resource-group $RG --nsg-name $NSG --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
2. Deploy troubleshooting stack with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario02/main.json --parameters location=$LOC nsgName=nsg-az104-d5-troubleshoot vnetName=vnet-az104-d5-troubleshoot subnetName=snet-app`
   - Note: this deployment already injects `deny-http-lab` (TCP/80 Deny, priority 200), so skip manual problem injection unless you are following a Portal-only setup.
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario02/main.json --parameters location=$LOC nsgName=nsg-az104-d5-troubleshoot vnetName=vnet-az104-d5-troubleshoot subnetName=snet-app`
4. Troubleshoot via logs/deployments:
   - `az monitor activity-log list --resource-group $RG --max-events 30 --output table`
   - `az deployment group list --resource-group $RG --output table`
5. Inspect restrictive NSG rule:
   - `az network nsg rule list --resource-group $RG --nsg-name nsg-az104-d5-troubleshoot --output table`

## Validation
- At least one concrete issue is captured (for example injected `deny-http-lab` rule or other restrictive NSG behavior).
- Root cause details is recorded from **Activity log** (operation history) and NSG configuration state.
- No **Failed** Activity Log entry is also a valid outcome for this scenario if NSG write operations succeeded.
- A specific fix is applied (for example corrected NSG rule priority/protocol/port or subnet/NSG association).
- Post-fix check confirms expected state (Portal shows updated configuration and no new failure for the same operation).
- Optional deep-dive path: before/after HTTP test details confirms real traffic impact of deny vs allow.
- Final troubleshooting note includes: problem, details, fix, and validation result.

## Complete Cleanup
- `az network vnet delete --resource-group rg-az104 --name vnet-az104-d5-troubleshoot`
- `az network nsg delete --resource-group rg-az104 --name nsg-az104-d5-troubleshoot`
