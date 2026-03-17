# Day 3 - Lab 2: App Roles and API Permissions

## Context (Why this lab matters)
- Permission modeling and consent flow are core SC-300 tasks.

## Objective
Configure API permissions and validate consent requirements.

## Scenario
An app needs Graph access with least-privilege design and documented consent behavior.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> App registrations -> app-sc300-baseline.
2. Open API permissions -> Add a permission -> Microsoft Graph.
3. Add User.Read (Delegated).
4. Add one admin-consent-required permission for study comparison.
5. Review and record which permissions require admin consent.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"
Get-MgApplication -Filter "displayName eq 'app-sc300-baseline'"
```

## Azure CLI Path (Method #3)
```bash
APP_ID=$(az ad app list --display-name app-sc300-baseline --query "[0].appId" -o tsv)
az ad app permission list --id "$APP_ID"
az ad app permission add --id "$APP_ID" --api 00000003-0000-0000-c000-000000000000 --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope
```

## IaC Templates (Optional - ARM)
- App permissions and consent are identity operations handled via Graph.

## Prerequisites
- App registration from Day 3 Lab 1 exists.

## Sequence Notes
- Start with least access.
- Avoid broad permissions unless this is a controlled test tenant.

## Validation
- Permission list shows expected entries.
- Consent status is captured.

## Complete Cleanup
- Remove any high-privilege test permission not needed after lab.
