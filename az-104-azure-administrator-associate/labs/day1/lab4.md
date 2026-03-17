# Day 1 - Lab 4: Identity Governance Extras (Users, Guests, Groups, Licensing)

## Context (Why this lab matters)
- AZ-104 identity questions often mix Entra identity lifecycle tasks with Azure RBAC tasks.
- This lab closes common identity-admin gaps: external users, group-based access, and licensing checks.
- You should finish with clear details of identity configuration state and role scope interpretation.

## Objective
Practice Entra identity governance basics (users/groups/guests/license visibility) and map identities to Azure RBAC at RG scope.

## Scenario
You are asked to validate identity governance readiness for a small project team.
You must onboard identities, review identity configuration basics, and prove resource access via RBAC scope.

## Important Licensing Note (Personal Subscription)
- Dynamic group membership requires Microsoft Entra ID P1 (or higher).
- Some identity features can depend on tenant policy and licensing configuration.
- If a feature is unavailable, note down the blocker and continue with the remaining steps.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Microsoft Entra ID** (Default Directory) -> **Manage** -> **Users**.
2. Create one test cloud user (e.g. "student" ) and (optional) invite one guest user.
3. Open **Microsoft Entra ID** -> **Groups** -> **New group** and create `az104-d1-identity-review`.
4. Open the group -> **Members** -> **Add members**, then manually add the created user/guest identities as member of this group.
5. Open **Microsoft Entra ID** -> **Manage** -> **Licenses** and review tenant license visibility and assignment options. If you see "You don't have access" , then just skip this step.
6. Open **Resource groups** -> **rg-az104** -> **Access control (IAM)** -> **+Add**  -> **Add role assignment**, and assign `Reader` role to `az104-d1-identity-review` group.
7. Open **Check access** for one test identity (e.g. "student") and confirm effective role/scope.
8. Note down details: user/guest presence, group membership, license visibility, RBAC role/scope.
9. Optional extension: run `labs/optional-labs/lab-sspr-review.md` for SSPR-specific review.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az ad signed-in-user show --output table
az ad group list --filter "displayName eq 'az104-d1-identity-review'" --output table
az role assignment list --resource-group $RG --query "[?roleDefinitionName=='Reader'].{Principal:principalName,Role:roleDefinitionName,Scope:scope}" --output table
```

## Optional Deep-Dive Exercise (Identity Scope Trap Proof)
Use this only if you want explicit exam-trap details.

### Create
1. Keep one test identity in Entra group `az104-d1-identity-review`.
2. Ensure group has `Reader` at RG scope.

### Validate
1. Confirm the identity can read RG resources.
2. Confirm the same identity cannot perform write operations at RG scope.
3. Note down one example that proves Entra identity configuration and Azure RBAC authorization are separate control planes.

### Cleanup
1. Remove temporary RBAC assignment created only for this lab.
2. Remove temporary users/guests/group if they were created only for this practice.

## IaC Templates (Optional - Later)
- Day 1 identity/governance labs are Portal-first by design.
- Optional ARM scenario templates can be added later if needed.

## Prerequisites
- `rg-az104` in `northeurope`
- Permission to manage Entra users/groups in the current tenant
- If RG is missing: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: complete Portal steps first; use CLI helper for verification.
- If tenant restrictions block a step, note down the blocker and continue with remaining objective areas.
- Keep this lab simple; avoid creating large numbers of test identities.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
GROUP=az104-d1-identity-review
az account show --output table
az group show --name $RG --output table
az ad group list --filter "displayName eq '$GROUP'" --output table
az role assignment list --resource-group $RG --query "[?principalName!=null].{Principal:principalName,Role:roleDefinitionName,Scope:scope}" --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && GROUP=az104-d6-identity-review && az account show --output table && az group create --name $RG --location northeurope --output table && az group show --name $RG --output table && az ad group list --filter "displayName eq '$GROUP'" --output table && az role assignment list --resource-group $RG --query "[?principalName!=null].{Principal:principalName,Role:roleDefinitionName,Scope:scope}" --output table
```

## Exercise
1. Create one test user + optional guest user in Entra.
2. Create group `az104-d1-identity-review` and add members.
3. Review licensing settings in Portal.
4. Assign `Reader` on `rg-az104` to the group.
5. Validate effective access in Portal and CLI.

## Validation
- At least one group-based RBAC assignment is confirmed at RG scope.
- Licensing visibility was reviewed and documented.
- Identity IAM vs RBAC IAM distinction is captured in your notes.

## Complete Cleanup
- Remove temporary lab role assignment:
  - `az role assignment list --resource-group rg-az104 --output table`
- Remove temporary test identities/groups created only for this lab.
