# Day 3 - Lab 1: App Registration and Enterprise Application Baseline

## Context (Why this lab matters)
- SC-300 requires you to distinguish app registrations from enterprise applications (service principals).

## Objective
Create an app registration, verify service principal creation, and assign app ownership.

## Scenario
A new internal app needs tenant identity setup and managed ownership.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> App registrations -> New registration.
2. Name the app app-sc300-baseline and keep single-tenant audience.
3. Save and note Application (client) ID and Directory (tenant) ID.
4. Open Enterprise applications and confirm matching service principal exists.
5. Add one owner for clear ownership.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"
$app = New-MgApplication -DisplayName "app-sc300-baseline" -SignInAudience "AzureADMyOrg"
New-MgServicePrincipal -AppId $app.AppId
Get-MgServicePrincipal -Filter "appId eq '$($app.AppId)'"
```

## Azure CLI Path (Method #3)
```bash
az ad app create --display-name app-sc300-baseline --sign-in-audience AzureADMyOrg
APP_ID=$(az ad app list --display-name app-sc300-baseline --query "[0].appId" -o tsv)
az ad sp create --id "$APP_ID"
az ad sp list --filter "appId eq '$APP_ID'" --output table
```

## IaC Templates (Optional - ARM)
- Entra app objects are Graph-driven and not fully ARM-native.

## Prerequisites
- Permission to create app registrations and service principals.

## Sequence Notes
- Register app first, then verify enterprise app object.
- Keep object IDs in notes for Day 3 Lab 2 and Lab 4.

## Validation
- App registration exists.
- Service principal exists and maps to the same appId.
- Owner assignment is visible.

## Complete Cleanup
- Keep app until Day 3 completion, then remove if no longer needed.
