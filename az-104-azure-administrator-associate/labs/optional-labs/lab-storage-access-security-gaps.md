# Day 8 - Lab 1: Storage Access and Security Gaps

## Objective
Cover remaining storage objectives: SAS tokens, stored access policies, access keys, firewall/network rules, and identity-based access for Azure Files.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Storage accounts** -> select a lab storage account in `rg-az104`.
2. Open **Security + networking** -> **Access keys** and review key1/key2 rotation workflow.
3. Open **Shared access signature** and generate one short-lived SAS token for Blob service.
4. Open **Containers** -> select/create one container and test SAS-based access (read/list as applicable).
5. Open **Networking** and switch from all networks to selected networks; add an allowed network path as available.
6. Open **File shares** and review identity-based access settings for Azure Files.
7. Note down what changed and what validated successfully.

## Azure CLI Helper
```bash
RG=rg-az104
SA=<storage-account-name>
az storage account show --resource-group $RG --name $SA --output table
az storage account keys list --resource-group $RG --account-name $SA --output table
az storage container list --account-name $SA --auth-mode login --output table
az storage account network-rule list --resource-group $RG --account-name $SA --output json
```

## Validation
- SAS token creation path is understood and tested.
- Access key management/rotation impact is documented.
- Storage network restriction state is verified.

## Complete Cleanup
- Revert temporary restrictive network settings if they block subsequent labs.
- Remove temporary SAS artifacts or notes containing secrets.
