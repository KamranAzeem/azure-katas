# Day 8 - Lab 2: Compute and App Platform Gaps

## Objective
Cover remaining compute/platform objectives: VM encryption/availability/move concepts, plus container and App Service advanced features.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> existing VM in `rg-az104` and review:
   - disk settings,
   - availability options,
   - encryption-related settings/blades.
2. Open VM **Move** workflow and walk through prerequisites (do not execute cross-subscription move unless intended).
3. Open **Container registries** and create/review one minimal ACR baseline.
4. Open **Container Instances** or **Container Apps** and run one minimal container deployment.
5. Open existing App Service and review:
   - deployment slots,
   - TLS/certificates,
   - custom domain mapping flow,
   - backup settings.
6. Note down which features are fully configured vs concept-reviewed only.

## Azure CLI Helper
```bash
RG=rg-az104
az vm list --resource-group $RG --output table
az acr list --resource-group $RG --output table
az container list --resource-group $RG --output table
az webapp list --resource-group $RG --output table
```

## Validation
- VM advanced admin paths are reviewed with clear notes.
- At least one container service path is exercised.
- App Service advanced feature locations are identified and documented.

## Complete Cleanup
- Delete temporary ACR/container resources created only for this lab.
