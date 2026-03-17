# Optional Lab: SSPR Policy Review (Entra)

## Context (Why this lab matters)
- SSPR visibility is useful for identity operations, but tenant licensing/policy constraints can block hands-on changes.
- This optional lab isolates SSPR-specific checks from the core identity baseline.

## Objective
Review Microsoft Entra SSPR configuration state and document operational implications for your tenant.

## Scenario
You need to confirm whether end users can self-reset passwords and whether registration requirements are configured.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Microsoft Entra ID** -> **Password reset**.
2. Review **Properties** (enabled scope: None/Selected/All).
3. Review **Authentication methods** and note required methods.
4. Review **Registration** settings and campaign/registration behavior.
5. Note down details: enabled scope, methods, registration policy, and any tenant/licensing blockers.

## Azure CLI Helper
```bash
az account show --output table
az ad signed-in-user show --output table
```

## Prerequisites
- Access to Microsoft Entra admin blades in your tenant.

## Sequence Notes
- This lab is review-focused; if tenant policy blocks edits, capture blocker details and continue.

## Quick Run (Azure CLI)
```bash
az account show --output table
az ad signed-in-user show --output table
```

### One-Liner (copy/paste)
```bash
az account show --output table && az ad signed-in-user show --output table
```

## Validation
- SSPR scope and registration settings are documented.
- At least one operational note is captured (for example, user impact or licensing dependency).

## Complete Cleanup
- None (review-only).
