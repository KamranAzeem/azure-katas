# Day 3 - Lab 4: Service Principal Credential Hygiene

## Context (Why this lab matters)
- SC-300 evaluates credential lifecycle controls for application identities.

## Objective
Create, review, and rotate service principal credentials.

## Scenario
Security review found long-lived credentials that must be rotated and documented.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> App registrations -> app-sc300-baseline.
2. Open Certificates and secrets.
3. Create client secret with short validity window.
4. Record secret name and expiration date.
5. Create replacement secret and retire previous test secret.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"
$app = Get-MgApplication -Filter "displayName eq 'app-sc300-baseline'"
Add-MgApplicationPassword -ApplicationId $app.Id -PasswordCredential @{displayName="sc300-short-lived";endDateTime=(Get-Date).AddDays(30)}
Get-MgApplication -ApplicationId $app.Id | Select-Object -ExpandProperty PasswordCredentials
```

## Azure CLI Path (Method #3)
```bash
APP_ID=$(az ad app list --display-name app-sc300-baseline --query "[0].appId" -o tsv)
az ad app credential reset --id "$APP_ID" --append --display-name sc300-short-lived --years 1
az ad app credential list --id "$APP_ID" --output table
```

## IaC Templates (Optional - ARM)
- Service principal secret lifecycle is Graph-driven.

## Prerequisites
- App registration exists from Day 3 Lab 1.

## Sequence Notes
- Use short-lived credentials for practice.
- Never expose generated secret values in logs.

## Validation
- Credential list reflects new credential and expected expiry.
- Rotation process is documented.

## Complete Cleanup
- Remove temporary secrets created for lab validation.
