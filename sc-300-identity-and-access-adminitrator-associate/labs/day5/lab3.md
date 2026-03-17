# Day 5 - Lab 3: Readiness Scorecard and Cleanup

## Context (Why this lab matters)
- Final readiness requires both technical competence and clean tenant hygiene.

## Objective
Score readiness by domain and complete cleanup.

## Scenario
You must submit readiness status and return the lab environment to a clean baseline.

## Direct Portal GUI Steps (Primary)
1. Score each domain from 1 to 5:
   - Identity lifecycle
   - Authentication and access management
   - Workload identities
   - Identity governance
2. List temporary objects created during SC-300 labs.
3. Remove or disable non-baseline objects.
4. Verify no temporary high-privilege assignments remain.

## PowerShell Path (Method #2)
- Use Microsoft Graph and Az modules to enumerate and remove temporary objects.

## Azure CLI Path (Method #3)
- Use az ad and az rest list commands to verify cleanup status.

## IaC Templates (Optional - ARM)
- Use ARM only for deleting supporting Azure resources if created.

## Prerequisites
- Access to all objects created during labs.

## Sequence Notes
- Keep baseline objects you intentionally plan to reuse.
- Document what was retained and why.

## Validation
- Scorecard completed.
- Cleanup checks pass and no orphaned privileged items remain.

## Complete Cleanup
- Final pass complete for users, groups, apps, policies, workflows, and Azure-side support resources.
