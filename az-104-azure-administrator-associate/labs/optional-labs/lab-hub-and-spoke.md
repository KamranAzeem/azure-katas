# Optional Lab: Hub-and-Spoke Network Design

## Context (Why this lab matters)
- Hub-and-spoke design is a core Azure networking pattern for centralizing shared services and routing.
- AZ-104 expects you to understand peering, address planning, and the operational details of network intent.
- This lab proves you can implement a minimal hub-and-spoke layout without costly gateways.

## Objective
Design and implement a minimal hub-and-spoke topology using VNet peering in `rg-az104`.

## Scenario
Your team needs a shared-services hub and a workload spoke.
You must build the topology and capture details that peering is connected and intentional.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Virtual networks** -> **Create**.
2. Create hub VNet `vnet-az104-d7-hub` with address space `10.80.0.0/16` and subnet `snet-shared` (`10.80.1.0/24`).
3. Create spoke VNet `vnet-az104-d7-spoke` with address space `10.81.0.0/16` and subnet `snet-workload` (`10.81.1.0/24`).
4. Open hub VNet -> **Peerings** -> **Add**:
   - Name: `peer-hub-to-spoke`
   - Remote VNet: `vnet-az104-d7-spoke`
   - Allow VNet access: **Yes**
5. Open spoke VNet -> **Peerings** -> **Add**:
   - Name: `peer-spoke-to-hub`
   - Remote VNet: `vnet-az104-d7-hub`
   - Allow VNet access: **Yes**
6. Verify both peerings show **Connected**.
7. Note down details: address spaces, peering names, and connection status.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az network vnet list --resource-group $RG --query "[].{Name:name,AddressSpace:addressSpace.addressPrefixes}" --output table
az network vnet peering list --resource-group $RG --vnet-name vnet-az104-d7-hub --output table
az network vnet peering list --resource-group $RG --vnet-name vnet-az104-d7-spoke --output table
```

## Optional Deep-Dive Exercise (Second Spoke)
Use this only if you want multi-spoke details.

### Create
1. Create a second spoke VNet `vnet-az104-d7-spoke2` with `10.82.0.0/16`.
2. Peer hub -> spoke2 and spoke2 -> hub (same settings as above).

### Validate
1. Confirm hub now has two peerings in **Connected** state.
2. Note down a note on why spokes do not peer to each other by default.

### Cleanup
1. Remove the optional second spoke VNet and its peerings if created only for this test.

## IaC Templates (Optional - Later)
- Optional labs are Portal-first by design.
- Optional ARM scenarios can be added if you want repeatable hub-and-spoke redeploys.

## Prerequisites
- `rg-az104` in `northeurope`
- If missing: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Keep address spaces non-overlapping.
- Use the smallest possible scope for this lab to reduce cost and time.
- Capture details right after peering so status is clear.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
HUB=vnet-az104-d7-hub
SPOKE=vnet-az104-d7-spoke
az account show --output table
az group show --name $RG --output table
az network vnet create --resource-group $RG --location $LOC --name $HUB --address-prefixes 10.80.0.0/16 --subnet-name snet-shared --subnet-prefixes 10.80.1.0/24
az network vnet create --resource-group $RG --location $LOC --name $SPOKE --address-prefixes 10.81.0.0/16 --subnet-name snet-workload --subnet-prefixes 10.81.1.0/24
az network vnet peering create --resource-group $RG --name peer-hub-to-spoke --vnet-name $HUB --remote-vnet $SPOKE --allow-vnet-access
az network vnet peering create --resource-group $RG --name peer-spoke-to-hub --vnet-name $SPOKE --remote-vnet $HUB --allow-vnet-access
az network vnet peering list --resource-group $RG --vnet-name $HUB --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && HUB=vnet-az104-d7-hub && SPOKE=vnet-az104-d7-spoke && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az network vnet create --resource-group $RG --location $LOC --name $HUB --address-prefixes 10.80.0.0/16 --subnet-name snet-shared --subnet-prefixes 10.80.1.0/24 && az network vnet create --resource-group $RG --location $LOC --name $SPOKE --address-prefixes 10.81.0.0/16 --subnet-name snet-workload --subnet-prefixes 10.81.1.0/24 && az network vnet peering create --resource-group $RG --name peer-hub-to-spoke --vnet-name $HUB --remote-vnet $SPOKE --allow-vnet-access && az network vnet peering create --resource-group $RG --name peer-spoke-to-hub --vnet-name $SPOKE --remote-vnet $HUB --allow-vnet-access && az network vnet peering list --resource-group $RG --vnet-name $HUB --output table
```

## Exercise
1. Create hub and spoke VNets with non-overlapping address spaces.
2. Configure bi-directional peering and validate connected state.
3. Capture a short design note: why the hub is central and what shared services could live there.

## Validation
- Hub and spoke VNets exist with expected address spaces.
- Both peering connections show **Connected**.
- Notes include a clear hub-and-spoke rationale.

## Complete Cleanup
- Remove temporary VNets and peerings created only for this lab.
