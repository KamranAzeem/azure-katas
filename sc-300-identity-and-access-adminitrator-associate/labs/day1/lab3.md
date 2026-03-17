# Day 1 - Lab 3: External Identity Guest Access Baseline

## Context (Why this lab matters)
- SC-300 includes B2B collaboration controls and external identity governance.

## Objective
Invite a guest, review external collaboration restrictions, and validate guest lifecycle state.

## Scenario
A partner consultant requires temporary tenant access.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Users -> New guest user.
2. Invite guest@example.com (or a controlled test mailbox).
3. Open External Identities -> External collaboration settings.
4. Review invite restrictions and guest permission defaults.
5. Track guest invitation status in Users blade.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "User.Invite.All","User.Read.All"
New-MgInvitation -InvitedUserEmailAddress "guest@example.com" -InviteRedirectUrl "https://myapps.microsoft.com" -SendInvitationMessage
Get-MgUser -Filter "userType eq 'Guest'" -Top 10
```

## Azure CLI Path (Method #3)
```bash
cat > /tmp/sc300-invite.json <<JSON
{"invitedUserEmailAddress":"guest@example.com","inviteRedirectUrl":"https://myapps.microsoft.com","sendInvitationMessage":true}
JSON
az rest --method POST --url https://graph.microsoft.com/v1.0/invitations --headers "Content-Type=application/json" --body @/tmp/sc300-invite.json
az rest --method GET --url "https://graph.microsoft.com/v1.0/users?$filter=userType eq 'Guest'"
```

## IaC Templates (Optional - ARM)
- Guest invitation workflow is Graph-driven and not ARM-native.

## Prerequisites
- Tenant must allow B2B invitations for your admin role.

## Sequence Notes
- Use a valid external mailbox you can monitor.
- Keep one guest for later governance labs if desired.

## Validation
- Guest user exists with userType set to Guest.
- Invitation object is created and state can be tracked.

## Complete Cleanup
- Delete guest user after testing if no longer needed.
