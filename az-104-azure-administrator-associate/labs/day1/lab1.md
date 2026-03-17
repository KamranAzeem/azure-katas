# Day 1 - Lab 1: RBAC Scope Audit (Shared Subscription)

## Context (Why this lab matters)
- AZ-104 frequently tests scope-aware access review and least-privilege reasoning.
- This lab builds confidence reading effective permissions at RG scope versus inherited scopes.
- You should finish with clear understanding of who has access and why, backed by observable results.

## Objective
Validate and review role assignments at resource group scope for `rg-az104`.

## Scenario
You are asked to confirm whether access to `rg-az104` is direct or inherited.
Your output is a short access summary showing principal, role, and scope observations.

## AZ-104 Expectation Note
- Azure CLI (`az`) is part of AZ-104 expectations and is used throughout this course for validation and troubleshooting.
- Portal + CLI competency is the primary goal for exam readiness in these labs.
- ARM template authoring is included here as optional reinforcement, not as a required Day 1 skill.

## Important Licensing Note (Personal Subscription)
- Dynamic group membership (auto membership) in Microsoft Entra ID requires Microsoft Entra ID P1 (or higher).
- If your tenant does not have P1, use manual group membership for this lab and keep the RBAC validation steps unchanged.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Resource groups** -> **rg-az104**.
2. Go to the newly created resource, and open **Access control (IAM)** -> **Role assignments**.
3. Look at the list and observe which users/groups are assigned which roles, and whether those assignments are directly on this resource group or inherited from the subscription level above it.
4. Open **Check access** ->  **View my access** for your signed-in user.
5. Look at the **Scope** column: if it shows the subscription path, your access is inherited; if it shows this resource group path, it's assigned directly here.

If you are using your personal account for Azure portal, and this is a fresh setup, then you will see two roles:
* A "Owner" Role , that has Scope set to "Subscription (Inherited)" - i.e. Inherited from Subscription level
* A "User Access Administrator" Role, that has Scope set to "Root (Inherited)" - i.e. Inherited from Root (management group) level (which is even a higher scope)

Neither is directly assigned to the "rg-az104" resource group - both are inherited from higher levels.

## Azure CLI Helper
```bash
RG=rg-az104
ASSIGNEE_OID=$(az ad signed-in-user show --query id -o tsv)
az account show --output table
az group show --name $RG --output table
az role assignment list --resource-group $RG --output table # Empty if nobody assigned a role directly to this RG.
az role assignment list --resource-group $RG --assignee-object-id "$ASSIGNEE_OID" --include-inherited --fill-principal-name false --query "[].{Role:roleDefinitionName,Scope:scope,PrincipalId:principalId}" --output table
```

## Optional Deep-Dive Exercise (RBAC Scope Proof)
Use this only if you want deeper details of inherited permissions.

- Scope note: this deep-dive is Azure RBAC IAM ("who" can do "what" at resource scope), not Entra identity lifecycle management.

### Create
1. Pick one principal (user/group/service principal) with known access.
2. Identify one assignment at subscription scope and one at RG scope (if available).

### Validate
1. Run:
   - `az role assignment list --assignee <principal-upn-or-object-id> --include-inherited --all --output table`
2. Confirm which effective permissions are inherited versus directly assigned.
3. Check or verify various settings related to role, scope, and inheritance.

### Cleanup
1. Remove any temporary assignment created only for this optional check.
2. Keep required baseline role assignments unchanged.

## Optional IAM Identity Mini Exercise (Users, Groups, Membership)
Use this if you want a simple IAM identity practice before moving on.

- Scope note: this section focuses on Entra identity IAM (users, groups, and membership), then maps those identities to Azure RBAC roles.

### Create
1. Open Azure Portal -> **Microsoft Entra ID** -> **Users** -> **New user** and create two test users (or invite two guest users).
2. Open Azure Portal -> **Microsoft Entra ID** -> **Groups** -> **New group** and create:
   - `az104-rg-readers`
   - `az104-rg-contributors`
3. For each group, set membership:
   - Default path: **Membership type = Assigned**, then **Members** -> add one test user to each group.
   - Optional path (only if P1 is available): **Membership type = Dynamic User** and define a simple rule.
4. Open **Resource groups** -> **rg-az104** -> **Access control (IAM)** -> **Add role assignment** and assign:
   - `Reader` to `az104-rg-readers`
   - `Contributor` to `az104-rg-contributors`

### Validate
1. Open Azure Portal -> **Resource groups** -> **rg-az104** -> **Access control (IAM)** -> **Check access**.
2. Search each test user and verify effective access:
   - Reader member can view resources.
   - Contributor member can create/update resources in RG scope.
3. Run CLI proof for one test identity:
   - `az role assignment list --resource-group rg-az104 --assignee <user-upn-or-object-id> --include-inherited --output table`
4. Note down principal, group, role, and scope details.

### Cleanup
1. Remove test role assignments from `rg-az104`.
2. Remove test users/guests and test groups if they were created only for this exercise.

## Additional IAM Scenarios (Choose for Lab 1)
- Group-based RBAC at RG scope (easiest, most exam-relevant).
- Inherited access test (subscription role vs RG role precedence).
- Entra directory role vs Azure RBAC comparison (conceptual, useful for exam traps).

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day1/scenario01-rbac-rg-scope/main.json`

## Prerequisites
- Azure CLI logged in: `az login`
- Active subscription selected
- Shared resource group exists in North Europe: `rg-az104`
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
SUB_ID=$(az account show --query id -o tsv)
ASSIGNEE_OID=$(az ad signed-in-user show --query id -o tsv)
az account show --output table
az group create --name $RG --location $LOC --output table
az group show --name $RG --output table
az role assignment list --resource-group $RG --output table
az role assignment list --resource-group $RG --assignee-object-id "$ASSIGNEE_OID" --include-inherited --fill-principal-name false --query "[].{Role:roleDefinitionName,Scope:scope,PrincipalId:principalId}" --output table

# Fallback if no direct RG assignments are returned
RG_SCOPE="/subscriptions/$SUB_ID/resourceGroups/$RG"
az role assignment list --all --query "[?scope=='$RG_SCOPE' || starts_with(scope, '$RG_SCOPE/')].[principalName,roleDefinitionName,scope]" --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && SUB_ID=$(az account show --query id -o tsv) && ASSIGNEE_OID=$(az ad signed-in-user show --query id -o tsv) && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az role assignment list --resource-group $RG --output table && az role assignment list --resource-group $RG --assignee-object-id "$ASSIGNEE_OID" --include-inherited --fill-principal-name false --query "[].{Role:roleDefinitionName,Scope:scope,PrincipalId:principalId}" --output table
```

## Exercise
1. Confirm subscription and RG:
   - `az account show --output table`
   - `az group show --name rg-az104 --output table`
2. List all role assignments at RG scope:
   - `SUB_ID=$(az account show --query id -o tsv)`
   - `az role assignment list --resource-group rg-az104 --output table`
   - If no direct RG assignments are shown: `RG_SCOPE="/subscriptions/$SUB_ID/resourceGroups/rg-az104" && az role assignment list --all --query "[?scope=='$RG_SCOPE' || starts_with(scope, '$RG_SCOPE/')].[principalName,roleDefinitionName,scope]" --output table`
3. Identify your effective role at RG scope:
   - `ASSIGNEE_OID=$(az ad signed-in-user show --query id -o tsv)`
   - `az role assignment list --resource-group rg-az104 --assignee-object-id "$ASSIGNEE_OID" --include-inherited --fill-principal-name false --query "[].{Role:roleDefinitionName,Scope:scope,PrincipalId:principalId}" --output table`

## Portal Path (Alternative)
1. Open Azure Portal and go to **Resource groups** > **rg-az104**.
2. Open **Access control (IAM)** > **Role assignments**.
3. Review direct assignments at RG scope and include inherited assignments from parent scopes.
4. Search for your signed-in identity and open **Check access** (or **View my access**) to confirm effective permissions.
5. Note down whether your effective access is direct at RG scope or inherited from subscription/management group.

## Validation
- You can explain which identity has which role at RG scope.
- You can distinguish Reader/Contributor/Owner capabilities.

## Complete Cleanup
- No resources created in this lab.
- No cleanup required.
