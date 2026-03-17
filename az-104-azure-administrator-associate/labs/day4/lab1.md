# Day 4 - Lab 1: Policy and Tags

## Context (Why this lab matters)
- Governance controls in AZ-104 are demonstrated through tags, policy scope, and compliance signals.
- This lab gives a practical enforcement/audit loop at RG scope.
- You should finish with visible tag posture and policy details.

## Objective
Apply concrete governance tags on `rg-az104`, assign a built-in tag policy at resource-group scope, and verify assignment/compliance visibility.

## Scenario
Governance controls are being audited on a shared resource group.
You must prove required tags and policy assignment/compliance signals are visible and actionable.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Resource groups** -> **rg-az104**.
2. Open **Tags** and add/update these values, then click **Save**:
   - `Environment=Lab`
   - `Owner=kamran`
   - `Course=AZ-104`
3. Open **Policies** and review current assignments/compliance at RG scope.
4. In **Policies**, click **Assign policy** and configure:
   - Scope: `rg-az104`
   - Policy definition: `Require a tag and its value on resources`
   - Parameters: `Tag Name = Environment`, `Tag Value = Lab`
   - Assignment name: keep default or set `pa-az104-tag-audit`
5. Create the assignment, wait 1-2 minutes, then refresh **Compliance**. You will see the new policy as "Not Started" under "Compliance State" column.
6. Note down final tag values and the policy assignment/compliance state.
    - Note down at minimum:
       - RG tags: `Environment=Lab`, `Owner=kamran`, `Course=AZ-104`
       - Policy assignment: `Require a tag and its value on resources` at RG scope (for example `pa-az104-tag-audit`)
       - Compliance state shown in Portal (for example `Not started` initially)
    - Example note: `Tags=Environment:Lab,Owner:kamran,Course:AZ-104; Assignment=pa-az104-tag-audit; Compliance=Not started`.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --query tags --output table
az policy assignment list --scope $(az group show -n $RG --query id -o tsv) --output table
```

## Optional Deep-Dive Exercise (Policy Enforcement Proof)
Use this only if you want explicit policy-evaluation details.

### Create
1. Keep assignment `Require a tag and its value on resources` at RG scope.
2. Attempt to create one temporary resource without required `Environment=Lab` tag.

### Validate
1. Capture actual behavior:
   - Denied at create time, or
   - Created but marked non-compliant (audit path).
2. Confirm details in Portal compliance view and/or CLI state query.
3. Note down assignment name, scope, and compliance/evaluation result.

### Recovery / Reset
1. Add missing required tag to temporary resource (if created).
2. Trigger scan and verify compliance transitions:
   - `az policy state trigger-scan --resource-group rg-az104`

### Cleanup
1. Delete temporary test resource and optional temporary assignment (if created only for this test).
2. Keep baseline tags/policy needed by the main lab path.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day4/scenario01/main.json`

## Prerequisites
- `rg-az104` in `northeurope`
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario01/main.json --parameters location=$LOC environmentTag=Lab ownerTag=kamran courseTag=AZ-104
az group show --name $RG --query tags --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario01/main.json --parameters location=$LOC environmentTag=Lab ownerTag=kamran courseTag=AZ-104 && az group show --name $RG --query tags --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
2. Deploy tags with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario01/main.json --parameters location=$LOC environmentTag=Lab ownerTag=kamran courseTag=AZ-104`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario01/main.json --parameters location=$LOC environmentTag=Lab ownerTag=kamran courseTag=AZ-104`
4. Optional: include policy assignment:
   - `POLICY_ID=$(az policy definition list --query "[?displayName=='Require a tag and its value on resources'].id | [0]" -o tsv)`
   - `az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario01/main.json --parameters location=$LOC policyDefinitionId=$POLICY_ID policyTagName=Environment`

## Validation
- Policy assignment exists at RG scope.
- RG tags are visible.

## Optional Mini Exercise (Validate Policy Behavior)
Use one temporary resource to confirm the policy is actually evaluated.

1. In Portal, try creating a temporary storage account in `rg-az104` **without** tag `Environment=Lab`.
2. Observe behavior:
   - If creation is blocked with a policy error, this is a valid enforcement result.
   - If creation succeeds, open **Policies** -> **Compliance** and confirm non-compliance is reported for the assigned policy.
3. If resource creation succeeded, add tag `Environment=Lab` on that temporary resource and refresh compliance after 1-2 minutes.
4. Delete the temporary storage account used for this check.

Note: The exact behavior depends on policy effect (`Deny` vs `Audit`) in your assignment and evaluation timing.

## Complete Cleanup
- `az policy assignment delete --name pa-az104-tag-audit --scope $(az group show -n rg-az104 --query id -o tsv)`
- `az group update --name rg-az104 --remove tags.Owner`

### Cleanup Notes (Portal-created Assignment Names)
- If the assignment was created from Portal, its name may be auto-generated (not `pa-az104-tag-audit`).
- Find assignment names at RG scope:
   - `az policy state list --resource-group rg-az104 --query "[?policyAssignmentName!=null].{AssignmentName:policyAssignmentName,AssignmentScope:policyAssignmentScope}" --output table`
- Delete using the discovered assignment name and scope:
   - `az policy assignment delete --name <assignment-name> --scope <assignment-scope>`

### Cleanup Notes (Stale Compliance Records)
- After deleting an assignment, **Policies/Compliance** or `az policy state list` can still show old non-compliant records for a few minutes.
- Trigger a re-evaluation scan and wait 3-10 minutes:
   - `az policy state trigger-scan --resource-group rg-az104`
- Re-check latest states:
   - `az policy state list --resource-group rg-az104 --top 5 --query "[].{AssignmentName:policyAssignmentName,ComplianceState:complianceState,Timestamp:timestamp}" --output table`
