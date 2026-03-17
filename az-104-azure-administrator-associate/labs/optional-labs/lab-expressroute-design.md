# Optional Lab: ExpressRoute Design (Concept + Details)

## Context (Why this lab matters)
- ExpressRoute is a private connectivity option for predictable enterprise network paths.
- For AZ-104, this topic is typically assessed at design/selection level more than full provider-backed implementation.
- This optional lab helps reinforce decision-making without requiring a carrier circuit.

## Objective
Create a minimal ExpressRoute design plan and capture details for why and when it is preferred over VPN.

## Scenario
Your organization needs private connectivity between on-premises and Azure for a critical workload.
You must propose a workable ExpressRoute setup and justify the choices.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **ExpressRoute circuits** -> **Create**.
2. Select subscription and resource group `rg-az104`.
3. Enter name `er-az104-opt` and location `North Europe`.
4. Choose provider, peering location, bandwidth, SKU tier, and billing model.
5. Do not create the circuit yet; capture your selected values as details.
6. If an existing circuit is available in your subscription, open it and review:
   - Circuit status
   - Provider provisioning status
   - Peering types and state

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az network express-route list --resource-group $RG --query "[].{Name:name,Location:location,Tier:sku.tier,Family:sku.family,Provider:serviceProviderProperties.serviceProviderName}" --output table
az network express-route list --query "[].{Name:name,Location:location,Tier:sku.tier,Provider:serviceProviderProperties.serviceProviderName}" --output table
```

## Optional Deep-Dive Exercise (Provider Fit)
### Create
1. Open the ExpressRoute provider list in your target region.
2. Shortlist two providers and one peering location.

### Validate
1. Document one final choice and rationale (latency, provider availability, cost model, SLA expectations).

### Cleanup
1. No cleanup required for this design-only exercise.

## Prerequisites
- `rg-az104` in `northeurope`
- If missing: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Keep this lab design-first to avoid cost and provider dependencies.
- If your subscription allows real creation and you choose to proceed, monitor cost implications.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
az account show --output table
az network express-route list --resource-group $RG --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && az account show --output table && az network express-route list --resource-group $RG --output table
```

## Validation
- A complete ExpressRoute circuit plan is documented.
- Notes include a clear reason for choosing ExpressRoute vs VPN for the scenario.

## Complete Cleanup
- None for design-only mode.
