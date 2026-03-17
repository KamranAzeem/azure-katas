# Optional Lab: Private DNS Zone and VNet Link (Auto-Registration)

## Context (Why this lab matters)
- AZ-104 networking operations often include name-resolution design for private endpoints and VM name records.
- This lab reinforces Private DNS zone basics, VNet linking, and auto-registration behavior.
- You should finish with clear details of DNS zone linkage and registration settings.

## Objective
Create a Private DNS zone, link it to a VNet, and validate registration behavior in `rg-az104`.

## Scenario
Your team needs private name resolution for internal workloads.
You must configure a Private DNS zone, attach a VNet link, and confirm whether automatic record registration is enabled.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Private DNS zones** -> **Create**.
2. Use resource group `rg-az104`, and create zone `az104.internal`.
3. Open the created zone -> **Virtual network links** -> **Add**.
4. Create link name `link-az104-dns-vnet`.
5. Select target VNet (for example your Day 2 VNet) and set **Enable auto registration** as needed:
   - `Yes` when VMs in that VNet should auto-register A records.
   - `No` for spoke/consumer VNets where registration is not desired.
6. Save the link and verify link status is **Completed**.
7. Open **Record sets** and confirm baseline record set visibility.
8. Note down details: zone name, linked VNet, registration enabled state, and observed record behavior.

## Azure CLI Helper
```bash
RG=rg-az104
ZONE=az104.internal
LINK=link-az104-dns-vnet
az account show --output table
az network private-dns zone list --resource-group $RG --query "[].{Name:name,ResourceGroup:resourceGroup}" --output table
az network private-dns link vnet list --resource-group $RG --zone-name $ZONE --query "[].{Name:name,VirtualNetwork:virtualNetwork.id,Registration:registrationEnabled,State:virtualNetworkLinkState}" --output table
az network private-dns record-set a list --resource-group $RG --zone-name $ZONE --output table
```

## Optional Deep-Dive Exercise (Registration Proof)
### Create
1. Ensure one test VM exists in the linked VNet.
2. Set registration enabled to `Yes` on the VNet link.

### Validate
1. Confirm an A record appears for the VM host name in the private zone.
2. Change registration to `No` and document expected behavior for new/updated records.

### Cleanup
1. Remove only temporary test records/links created for this exercise.

## Prerequisites
- `rg-az104` in `northeurope`
- At least one existing VNet to link
- If RG is missing: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Auto-registration is only supported for one registration-enabled link per VNet for the zone.
- Keep naming simple and deterministic for easier validation.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
ZONE=az104.internal
LINK=link-az104-dns-vnet
VNET=vnet-az104-d2
az account show --output table
az group show --name $RG --output table
az network private-dns zone create --resource-group $RG --name $ZONE
az network private-dns link vnet create --resource-group $RG --zone-name $ZONE --name $LINK --virtual-network $VNET --registration-enabled true
az network private-dns link vnet list --resource-group $RG --zone-name $ZONE --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && ZONE=az104.internal && LINK=link-az104-dns-vnet && VNET=vnet-az104-d2 && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az network private-dns zone create --resource-group $RG --name $ZONE && az network private-dns link vnet create --resource-group $RG --zone-name $ZONE --name $LINK --virtual-network $VNET --registration-enabled true && az network private-dns link vnet list --resource-group $RG --zone-name $ZONE --output table
```

## Validation
- Private DNS zone exists in `rg-az104`.
- VNet link exists and `Registration` state is documented.
- Notes include when to enable versus disable auto-registration.

## Complete Cleanup
- Remove temporary link and zone only if created for this practice:
  - `az network private-dns link vnet delete --resource-group rg-az104 --zone-name az104.internal --name link-az104-dns-vnet --yes`
  - `az network private-dns zone delete --resource-group rg-az104 --name az104.internal --yes`
