# AZ-104 Labs Quick Run Cheatsheet

Use this page as a fast launcher to each lab's **Quick Run (Azure CLI)** section.

## Day 1
- [Day 1 - Lab 1](day1/lab1.md#quick-run-azure-cli)
- [Day 1 - Lab 2](day1/lab2.md#quick-run-azure-cli)
- [Day 1 - Lab 3](day1/lab3.md#quick-run-azure-cli)
- [Day 1 - Lab 4](day1/lab4.md#quick-run-azure-cli)

### Day 1 RBAC note
If `az role assignment list --scope ...` returns `MissingSubscription`, use:

```bash
RG=rg-az104
ASSIGNEE_OID=$(az ad signed-in-user show --query id -o tsv)
az role assignment list --resource-group $RG --output table
az role assignment list --resource-group $RG --assignee-object-id "$ASSIGNEE_OID" --include-inherited --fill-principal-name false --query "[].{Role:roleDefinitionName,Scope:scope,PrincipalId:principalId}" --output table
```

## Day 2
- [Day 2 - Lab 1](day2/lab1.md#quick-run-azure-cli)
- [Day 2 - Lab 2](day2/lab2.md#quick-run-azure-cli)
- [Day 2 - Lab 3](day2/lab3.md#quick-run-azure-cli)
- [Day 2 - Lab 4](day2/lab4.md#quick-run-azure-cli)
- [Day 2 - Lab 5](day2/lab5.md#quick-run-azure-cli)

## Day 3
- [Day 3 - Lab 1](day3/lab1.md#quick-run-azure-cli)
- [Day 3 - Lab 2](day3/lab2.md#quick-run-azure-cli)
- [Day 3 - Lab 3](day3/lab3.md#quick-run-azure-cli)

## Day 4
- [Day 4 - Lab 1](day4/lab1.md#quick-run-azure-cli)
- [Day 4 - Lab 2](day4/lab2.md#quick-run-azure-cli)
- [Day 4 - Lab 3](day4/lab3.md#quick-run-azure-cli)
- [Day 4 - Lab 4](day4/lab4.md#quick-run-azure-cli)

## Day 5
- [Day 5 - Lab 1](day5/lab1.md#quick-run-azure-cli)
- [Day 5 - Lab 2](day5/lab2.md#quick-run-azure-cli)
- [Day 5 - Lab 3](day5/lab3.md#quick-run-azure-cli)

## Optional Labs
- [Optional - Hub-and-Spoke Network Design](optional-labs/lab-hub-and-spoke.md#quick-run-azure-cli)
- [Optional - ExpressRoute Design](optional-labs/lab-expressroute-design.md#quick-run-azure-cli)
- [Optional - SSPR Policy Review](optional-labs/lab-sspr-review.md#quick-run-azure-cli)
- [Optional Labs Index](optional-labs/README.md)
- [Optional - Storage Access and Security Gaps](optional-labs/lab-storage-access-security-gaps.md#azure-cli-helper)
- [Optional - Compute and App Platform Gaps](optional-labs/lab-compute-app-platform-gaps.md#azure-cli-helper)
- [Optional - Advanced Networking Gaps](optional-labs/lab-advanced-networking-gaps.md#azure-cli-helper)
- [Optional - Monitor and Backup/DR Gaps](optional-labs/lab-monitor-backup-dr-gaps.md#azure-cli-helper)
- [Optional - Private DNS Zone and VNet Link](optional-labs/lab-private-dns-zone-vnet-link.md#quick-run-azure-cli)

## Optional one-liner precheck
```bash
az group create --name rg-az104 --location northeurope --output table
az account show --output table
az group show --name rg-az104 --output table
```