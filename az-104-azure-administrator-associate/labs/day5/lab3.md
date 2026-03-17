# Day 5 - Lab 3: Readiness Scorecard + Cleanup Wrap

## Context (Why this lab matters)
- This lab closes Day 5 simulation by converting results into a concrete readiness scorecard.
- You will verify end-state inventory, remove non-required resources, and keep only intentional baseline assets.
- A clean subscription state improves repeatability for targeted remediation labs and optional extensions.

## Objective
Complete cleanup in `rg-az104`, verify expected end-state inventory, and note down a readiness scorecard with next remediation targets.

## Scenario
Day 5 simulation is complete and your next step is targeted gap closure.
You inventory what remains, remove non-required resources, and document a clean end state plus next-priority objectives.

## Resume from Day 5 Lab 2 (Recommended)
If you are moving directly from Lab 2:
- Primary cleanup targets:
   - `vnet-az104-d5-troubleshoot`
   - `nsg-az104-d5-troubleshoot`
- Verify they are deleted before broader cleanup:
   - `az network vnet show --resource-group rg-az104 --name vnet-az104-d5-troubleshoot --output table`
   - `az network nsg show --resource-group rg-az104 --name nsg-az104-d5-troubleshoot --output table`
   - Expected after delete: `ResourceNotFound` for both.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Resource groups** -> **rg-az104**.
2. Open **Overview** and review current resource count.
3. Open **Resources** list and sort/group by **Type** and **Name**.
   - Identify resources created for temporary labs (for example Day 5 troubleshoot/final-review items).
4. Delete non-required lab resources from the resource list.
   - Select resource -> **Delete** -> confirm name -> **Delete**.
   - Keep only baseline resources you intentionally retain for future labs.
5. Open **Activity log** and confirm delete operations complete successfully.
6. Open **Tags** on the resource group and review cleanup markers.
   - If `CleanupCandidate` (or other temporary marker tags) exists, remove it and save.
7. Re-open **Overview/Resources** and confirm final expected-state inventory.
8. Note down wrap-up notes (minimum):
   - What was deleted
   - What was intentionally retained
   - Final tag posture
   - Final resource count/state

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az resource list --resource-group $RG --output table
az group show --name $RG --query tags --output table
az resource list --resource-group $RG --query "[?name=='vnet-az104-d5-troubleshoot' || name=='nsg-az104-d5-troubleshoot'].{Type:type,Name:name}" --output table
```

## Optional Deep-Dive Exercise (Cleanup Details Proof)
Use this only if you want stronger cleanup audit details.

### Create
1. Capture pre-cleanup inventory snapshot:
   - `az resource list --resource-group rg-az104 --query "[].{Type:type,Name:name}" --output table`
2. Export/save the snapshot in your notes.

### Validate
1. Execute cleanup actions from this lab.
2. Capture post-cleanup inventory using the same command.
3. Compare before/after and note down exactly what changed.

### Recovery / Reset
1. If you removed a resource by mistake, re-create it from the relevant lab template.
2. Re-run post-cleanup inventory and confirm intended end state.

### Cleanup
1. Remove temporary snapshot files if they include sensitive naming/context.
2. Keep final notes with only required summary details.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day5/scenario03/main.json`

## Prerequisites
- Inventory of resources created during labs
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario03/main.json --parameters location=$LOC cleanupTagValue=true
az resource list --resource-group $RG --output table
az group show --name $RG --query tags --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario03/main.json --parameters location=$LOC cleanupTagValue=true && az resource list --resource-group $RG --output table && az group show --name $RG --query tags --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
2. Apply cleanup marker tag with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario03/main.json --parameters location=$LOC cleanupTagValue=true`
3. Or apply with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario03/main.json --parameters location=$LOC cleanupTagValue=true`
4. List all resources in shared RG:
   - `az resource list --resource-group $RG --output table`
5. Confirm cleanup tag:
   - `az group show --name $RG --query tags --output table`
6. Delete remaining lab resources by workload grouping.
7. Confirm empty/expected-state RG inventory.

### Fast Transition Step (Lab 2 -> Lab 3)
Run this first when you finish Day 5 Lab 2:
```bash
RG=rg-az104
az network vnet delete --resource-group $RG --name vnet-az104-d5-troubleshoot
az network nsg delete --resource-group $RG --name nsg-az104-d5-troubleshoot
az resource list --resource-group $RG --query "[?name=='vnet-az104-d5-troubleshoot' || name=='nsg-az104-d5-troubleshoot'].{Type:type,Name:name}" --output table
```

## Validation
- Resource list in `rg-az104` matches intended end state (only intentionally retained baseline assets remain).
- Temporary Day 5 resources (final-review/troubleshoot artifacts) are deleted or explicitly documented as retained.
- Day 5 Lab 2 targets `vnet-az104-d5-troubleshoot` and `nsg-az104-d5-troubleshoot` are removed.
- Cost-impacting resources (for example VMs, load balancers, public IPs, extra workspaces/storage accounts) are removed unless intentionally kept.
- Resource group tag review confirms temporary cleanup markers are removed when cleanup is complete.
- Activity log shows successful delete/update operations for cleanup actions performed in this lab.

## Complete Cleanup
- If full cleanup is allowed:
  - `az resource list --resource-group rg-az104 --query "[].id" -o tsv`
  - Delete resources individually or remove RG only if approved.
- Remove cleanup marker tag when done:
   - `az group update --name rg-az104 --remove tags.CleanupCandidate`
