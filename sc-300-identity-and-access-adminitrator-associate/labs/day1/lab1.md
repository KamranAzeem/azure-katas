# Day 1 - Lab 1: User and Group Lifecycle Baseline

## Context (Why this lab matters)
- SC-300 tests practical identity lifecycle administration in Microsoft Entra.
- This baseline is reused across later authentication, governance, and workload identity labs.

## Objective
Create cloud users and security groups, assign membership, and verify object state.

## Scenario
A project team is being onboarded and requires identity objects that can later be scoped by policy.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> Microsoft Entra ID -> Users -> New user.
2. Create two cloud users:
   - sc300.user01@<tenant>.onmicrosoft.com
   - sc300.user02@<tenant>.onmicrosoft.com
3. Open Microsoft Entra ID -> Groups -> New group.
4. Create security groups:
   - sc300-helpdesk
   - sc300-app-users
5. Add sc300.user01 to sc300-helpdesk.
6. Add sc300.user02 to sc300-app-users.
7. Open each user and group and confirm account is enabled and membership is visible.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","Directory.ReadWrite.All"
$tenantDomain = "<tenant>.onmicrosoft.com"
$pwd = @{ Password = "P@ssw0rd!1234" }

New-MgUser -DisplayName "SC300 User 01" -UserPrincipalName "sc300.user01@$tenantDomain" -MailNickname "sc300user01" -AccountEnabled -PasswordProfile $pwd
New-MgUser -DisplayName "SC300 User 02" -UserPrincipalName "sc300.user02@$tenantDomain" -MailNickname "sc300user02" -AccountEnabled -PasswordProfile $pwd

$g1 = New-MgGroup -DisplayName "sc300-helpdesk" -MailEnabled:$false -MailNickname "sc300helpdesk" -SecurityEnabled
$g2 = New-MgGroup -DisplayName "sc300-app-users" -MailEnabled:$false -MailNickname "sc300appusers" -SecurityEnabled

$u1 = Get-MgUser -Filter "userPrincipalName eq 'sc300.user01@$tenantDomain'"
$u2 = Get-MgUser -Filter "userPrincipalName eq 'sc300.user02@$tenantDomain'"

New-MgGroupMember -GroupId $g1.Id -DirectoryObjectId $u1.Id
New-MgGroupMember -GroupId $g2.Id -DirectoryObjectId $u2.Id
```

## Azure CLI Path (Method #3)
```bash
TENANT_DOMAIN="<tenant>.onmicrosoft.com"
az login

cat > /tmp/sc300-user01.json <<JSON
{"accountEnabled":true,"displayName":"SC300 User 01","mailNickname":"sc300user01","userPrincipalName":"sc300.user01@${TENANT_DOMAIN}","passwordProfile":{"forceChangePasswordNextSignIn":true,"password":"P@ssw0rd!1234"}}
JSON

cat > /tmp/sc300-user02.json <<JSON
{"accountEnabled":true,"displayName":"SC300 User 02","mailNickname":"sc300user02","userPrincipalName":"sc300.user02@${TENANT_DOMAIN}","passwordProfile":{"forceChangePasswordNextSignIn":true,"password":"P@ssw0rd!1234"}}
JSON

az rest --method POST --url https://graph.microsoft.com/v1.0/users --headers "Content-Type=application/json" --body @/tmp/sc300-user01.json
az rest --method POST --url https://graph.microsoft.com/v1.0/users --headers "Content-Type=application/json" --body @/tmp/sc300-user02.json

az ad group create --display-name sc300-helpdesk --mail-nickname sc300helpdesk
az ad group create --display-name sc300-app-users --mail-nickname sc300appusers
```

## IaC Templates (Optional - ARM)
- Identity objects (users/groups) are Graph-driven and not ARM-native.
- Use ARM only for supporting Azure resources in this track.

## Prerequisites
- Entra admin permissions to create users and groups.
- Microsoft Graph permissions consented for chosen method.

## Sequence Notes
- Complete Portal path first.
- Use PowerShell or CLI for verification and reuse.
- Keep naming consistent to simplify later labs.

## Quick Run (Azure CLI)
```bash
az ad user list --filter "startsWith(userPrincipalName,'sc300.user0')" --query "[].{UPN:userPrincipalName,Enabled:accountEnabled}" --output table
az ad group list --filter "startsWith(displayName,'sc300-')" --query "[].{Name:displayName,Id:id}" --output table
```

## Validation
- Both users exist and are enabled.
- Both groups exist.
- Group membership mapping is correct.

## Complete Cleanup
- Keep these objects for Day 2-Day 4 labs.
- Delete only if resetting the entire SC-300 practice run.
