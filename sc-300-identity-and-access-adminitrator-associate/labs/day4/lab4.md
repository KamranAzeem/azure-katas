# Day 4 - Lab 4: Lifecycle Workflows Baseline

## Context (Why this lab matters)
- Lifecycle workflows are key for joiner/mover/leaver process automation in SC-300.

## Objective
Create workflow from template and validate execution behavior.

## Scenario
HR needs automated user onboarding or offboarding actions.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Identity Governance -> Lifecycle workflows.
2. Create workflow from template.
3. Define user scope and trigger conditions.
4. Configure tasks and run test execution for one identity.
5. Review run history and outcome state.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "LifecycleWorkflows.ReadWrite.All"
Get-MgIdentityGovernanceLifecycleWorkflow
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/beta/identityGovernance/lifecycleWorkflows/workflows
```

## IaC Templates (Optional - ARM)
- Lifecycle workflows are Graph-governance resources and not ARM-native.

## Prerequisites
- Entra ID governance licensing and workflow permissions.

## Sequence Notes
- Use test identities for first execution.
- Review task dependencies before production-like rollout.

## Validation
- Workflow is created and enabled.
- At least one run is visible with task-level status.

## Complete Cleanup
- Disable or delete test workflows after completion.
