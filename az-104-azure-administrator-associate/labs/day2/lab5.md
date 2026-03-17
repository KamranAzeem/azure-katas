# Day 2 - Lab 5: Networking + Operations Gap Closure

## Context (Why this lab matters)
- AZ-104 networking objectives include peering, routes, and operational diagnostics beyond NSGs.
- This lab closes practical gaps with lightweight networking controls and monitoring tools.
- You should finish with clear details for route/peering intent and operational visibility.

## Objective
Validate core networking-operations gaps using VNet peering or UDR basics plus Network Watcher/Connection Monitor visibility in `rg-az104`.

## Scenario
You need to prove your ability to reason about network pathing and operational diagnostics.
You configure lightweight network controls and capture troubleshooting details.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Virtual networks** and confirm at least one lab VNet exists. If not, create one.
2. If two VNets exist, create peering between them. (Portal -> VNets -> Oslo Vnet -> Settings ->  Peerings -> Add) . You need to add two names for two directions. 
  - vnet-peering-oslo-to-asker 
  - vnet-peering-asker-to-oslo
  
  Also ensure the following are check-marked (enabled):
  - Allow 'vnet-day2-asker' to access 'vnet-day2-oslo'
  - Allow 'vnet-day2-oslo' to access 'vnet-day2-asker'

3. Configure one basic route-table association on a test subnet:
	 - On Main portal search for **Route tables** -> **+Create** and create `rt-az104-d2` in `rg-az104`.
	 - In `rt-az104-d2`, open "Settings" -> **Routes** -> **Add** and create a simple route:
		 - Destination Address range: `10.0.0.0/16` (the `oslo` VNet address space)
		 - Next hop type: `Virtual network gateway` (or `Internet` if gateway option is blocked)
	 - Go to **Subnets** inside the same route table and associate `rt-az104-d2` to the `asker` VNet `default` subnet.
4. Open **Network Watcher** and confirm region enablement for `North Europe`.
5. Inside "Network Watcher", open "Network diagnostic tools" -> **Connection troubleshoot** or **Connection monitor** and run one connectivity check between two endpoints.
	 - If VMs/endpoints are available: run one real connectivity test and capture the result.
	 - If no VMs/endpoints are available: use fallback validation instead:
		 - Confirm both peerings show **Connected** in `oslo` and `asker` VNet peering views.
		 - Confirm route table `rt-az104-d2` is associated to `asker/default` subnet.
		 - Confirm custom route to `10.0.0.0/16` is present in route table routes.
		 - Confirm **Network Watcher** is enabled for `North Europe`.
6. Note down details: peering/route config state, diagnostic test result (or no-VM fallback checks), and interpretation.
7. (Optional) Review Bastion and private endpoint blades conceptually if not deploying them in this lab.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az network vnet list --resource-group $RG --query "[].{Name:name,AddressSpace:addressSpace.addressPrefixes}" --output table
az network watcher list --query "[].{Name:name,Location:location,Enabled:provisioningState}" --output table
az network route-table list --resource-group $RG --output table
```

## Optional Deep-Dive Exercise (Routing Proof)
Use this only if you want explicit path-control details.

### Create
1. Create a route table `rt-az104-d2` with one custom route.
2. Associate route table to a test subnet.

### Validate
1. Confirm route table association in subnet configuration.
2. Run a connection diagnostic and compare result before/after association (if feasible).
3. Note down observed effect and any limitations in your environment.

### Cleanup
1. Remove temporary route table and association if created only for this test.
2. Keep only baseline network assets you intentionally retain.

## IaC Templates (Optional - Later)
- Day 2 labs are Portal-first by design.
- Optional ARM scenarios can be added for repeatable peering/route deployments.

## Prerequisites
- `rg-az104` in `northeurope`
- At least one VNet in the resource group (two VNets if testing real peering)
- If RG is missing: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: complete Portal steps first; use CLI helper for verification.
- If you cannot provision additional VNets due limits, run the diagnostics portion against existing resources.
- If no VMs/endpoints are present, use the fallback validation path in step 5 and note this limitation.
- Keep scope minimal and details-focused.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
VNET_A=oslo
VNET_B=asker
az account show --output table
az group show --name $RG --output table
# If these VNets do not already exist, create them with the lab-standard address spaces:
az network vnet create --resource-group $RG --location $LOC --name $VNET_A --address-prefixes 10.0.0.0/16 --subnet-name default --subnet-prefixes 10.0.0.0/24
az network vnet create --resource-group $RG --location $LOC --name $VNET_B --address-prefixes 10.1.0.0/16 --subnet-name default --subnet-prefixes 10.1.0.0/24
az network vnet peering create --resource-group $RG --name peer-a-to-b --vnet-name $VNET_A --remote-vnet $VNET_B --allow-vnet-access
az network vnet peering create --resource-group $RG --name peer-b-to-a --vnet-name $VNET_B --remote-vnet $VNET_A --allow-vnet-access
az network vnet peering list --resource-group $RG --vnet-name $VNET_A --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && VNET_A=oslo && VNET_B=asker && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az network vnet create --resource-group $RG --location $LOC --name $VNET_A --address-prefixes 10.0.0.0/16 --subnet-name default --subnet-prefixes 10.0.0.0/24 && az network vnet create --resource-group $RG --location $LOC --name $VNET_B --address-prefixes 10.1.0.0/16 --subnet-name default --subnet-prefixes 10.1.0.0/24 && az network vnet peering create --resource-group $RG --name peer-a-to-b --vnet-name $VNET_A --remote-vnet $VNET_B --allow-vnet-access && az network vnet peering create --resource-group $RG --name peer-b-to-a --vnet-name $VNET_B --remote-vnet $VNET_A --allow-vnet-access && az network vnet peering list --resource-group $RG --vnet-name $VNET_A --output table
```

## Exercise
1. Validate existing VNet/network-watcher state.
2. Run peering and/or route-table mini exercise.
3. Run one diagnostic check and interpret result.
	- If no endpoints are available, run fallback checks (peering connected, route-table association, route present, watcher enabled) and record the limitation.
4. Capture a short operations note with root-cause style reasoning.

## Validation
- At least one path-control construct (peering or route-table) is configured/validated.
- At least one network diagnostic output is captured and interpreted, or no-VM fallback checks are completed and documented.
- Notes include one troubleshooting conclusion backed by observed data.

## Complete Cleanup
- Remove temporary peering/VNets/route tables created only for this lab.
- Keep baseline assets required for ongoing study.
