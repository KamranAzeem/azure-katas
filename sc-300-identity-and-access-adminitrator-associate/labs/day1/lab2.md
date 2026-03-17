# Day 1 - Lab 2: Administrative Units and Scoped Administration

## Context (Why this lab matters)
- SC-300 expects delegated admin designs that avoid tenant-wide permissions.

## Objective
Create an administrative unit and apply scoped role delegation.

## Scenario
Regional operations should only manage users in their business segment.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Administrative units -> Add.
2. Create AU named au-sc300-nordics.
3. Add sc300.user01 and sc300.user02 as members.
4. Open Roles and administrators inside the AU.
5. Assign User Administrator (scoped to AU) to a test operator account.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "AdministrativeUnit.ReadWrite.All","RoleManagement.ReadWrite.Directory"
$au = New-MgDirectoryAdministrativeUnit -DisplayName "au-sc300-nordics" -Description "SC300 scoped admin unit"
$u1 = Get-MgUser -Filter "startsWith(userPrincipalName,'sc300.user01')"
New-MgDirectoryAdministrativeUnitMemberByRef -AdministrativeUnitId $au.Id -BodyParameter @{"@odata.id"="https://graph.microsoft.com/v1.0/directoryObjects/$($u1.Id)"}
```

## Azure CLI Path (Method #3)
```bash
cat > /tmp/sc300-au.json <<JSON
{"displayName":"au-sc300-nordics","description":"SC300 scoped admin unit","visibility":"Public"}
JSON
az rest --method POST --url https://graph.microsoft.com/v1.0/directory/administrativeUnits --headers "Content-Type=application/json" --body @/tmp/sc300-au.json
```

## IaC Templates (Optional - ARM)
- Administrative units are not ARM-native resources.

## Prerequisites
- Privileged role that can manage administrative units.

## Sequence Notes
- Add members before testing delegated scope.
- Use a non-global admin account for realistic validation.

## Validation
- AU exists with expected members.
- Scoped role assignment is visible under AU roles.

## Complete Cleanup
- Remove scoped role assignment if created only for this test.
