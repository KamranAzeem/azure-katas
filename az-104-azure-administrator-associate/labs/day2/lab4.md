# Day 2 - Lab 4: Compute + App Platform Gap Closure

## Context (Why this lab matters)
- AZ-104 compute coverage is broader than single-VM administration.
- This lab closes practical gaps around VM scale concepts and App Service fundamentals.
- You should finish with concrete details for compute and app-platform objective areas.

## Objective
Deploy and validate one lightweight VM scale set baseline and one basic App Service baseline in `rg-az104`.

Important: In this lab, VMSS and Web App are two separate exercises for comparison. They are not connected to each other.

## Scenario
Your team needs a scalable compute option and a managed web-hosting baseline.
You must provision both quickly, validate health, and note down differences versus single-VM operations.

## Direct Portal GUI Steps (Primary)
### Track A: VMSS Baseline (Independent)
1. Open Azure Portal -> **Virtual machine scale sets** -> **Create**.
2. Use resource group `rg-az104`, region `North Europe`, and name `vmss-az104-d2`.
3. Choose:
  - A low-cost Linux image and machine size.
  - Orchestration mode: `Uniform`
  - Scaling mode: `Manual`
  - Initial instance count: `1`
4. Open VMSS **Overview** and verify provisioning state and instance count.

### Track B: Web App Baseline (Independent)
(You may get an error about insufficient quota. In that case skip this.)
5. Open Azure Portal -> **App Services** -> **Create** -> **Web App**.
6. Create or reuse an App Service Plan (low-cost SKU), then create web app `app-az104-d2`.
   - Publish: `Code`
   - Runtime stack: `Node 20 LTS` (or nearest available Node LTS if Node 20 is not listed)
7. Open web app **Overview** and browse default endpoint to verify app availability.

### Compare and Note
8. Note down details: VMSS name/instances, App Service Plan tier, web app URL/status.
9. Note down one explicit comparison point (for example: VMSS = VM management responsibility, Web App = managed platform).

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az vmss list --resource-group $RG --query "[].{Name:name,Location:location,Sku:sku.name,Capacity:sku.capacity}" --output table
az appservice plan list --resource-group $RG --query "[].{Name:name,Sku:sku.name,Kind:kind}" --output table
az webapp list --resource-group $RG --query "[].{Name:name,State:state,Host:defaultHostName}" --output table
```

## Optional Deep-Dive Exercise (Scale vs Managed Platform Proof)
Use this only if you want stronger comparison details.

### Create
1. Scale VMSS capacity up by +1 instance. Use "Scaling" under "Availability + scale" blade.
2. Change App Service settings (for example Always On or basic app setting) and save.

### Validate
1. Confirm VMSS capacity change appears in instance list.
2. Confirm App Service setting change is applied and app remains reachable.
3. Note down which platform abstracts more OS management tasks.

### Cleanup
1. Revert temporary scale/config changes if not needed.
2. Keep baseline resources unless running full cleanup.

## IaC Templates (Optional - Later)
- Day 2 labs are Portal-first by design.
- Optional ARM scenarios can be added if you want repeatable redeploys.

## Prerequisites
- `rg-az104` in `northeurope`
- Sufficient quota for one small VMSS and one basic App Service
- If missing: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: complete Portal steps first; use CLI helper for verification.
- If quota blocks VMSS or App Service SKU, choose the smallest available alternative and continue.
- Keep deployment minimal; this is a gap-closure lab, not architecture build-out.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
PLAN=asp-az104-d2
APP=app-az104-d2$RANDOM
az account show --output table
az group show --name $RG --output table
az appservice plan create --resource-group $RG --name $PLAN --location $LOC --sku B1 --is-linux
az webapp create --resource-group $RG --plan $PLAN --name $APP --runtime "NODE:20-lts"
az appservice plan list --resource-group $RG --output table
az webapp list --resource-group $RG --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && PLAN=asp-az104-d2 && APP=app-az104-d2$RANDOM && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az appservice plan create --resource-group $RG --name $PLAN --location $LOC --sku B1 --is-linux && az webapp create --resource-group $RG --plan $PLAN --name $APP --runtime "NODE:20-lts" && az appservice plan list --resource-group $RG --output table && az webapp list --resource-group $RG --output table
```

## Exercise
1. Complete Track A (VMSS) and verify instance readiness.
2. Complete Track B (Web App) and verify endpoint readiness.
3. Compare operations model (patching/scale/management overhead) in your notes.

## Validation
- VMSS baseline is created and visible with expected capacity.
- Web App baseline is created and reachable.
- Notes include one clear difference between VM-based and managed app hosting operations.
- VMSS and Web App are treated as separate baselines (no integration required in this lab).

## Complete Cleanup
- Remove web app and plan if created for this lab only:
  - `az webapp list --resource-group rg-az104 --output table`
  - `az appservice plan list --resource-group rg-az104 --output table`
- Remove VMSS resources if created for this lab only.
