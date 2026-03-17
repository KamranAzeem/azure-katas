# SC-300 ARM Reinforcement Notes

SC-300 is primarily an identity/governance exam and many tasks are Microsoft Graph (tenant identity objects), not Azure Resource Manager resources.

Use ARM templates in this folder as supporting practice for Azure-side resources that commonly pair with SC-300 workflows (logging, managed identity hosts, Key Vault, and evidence storage).

## Important Limitation
- Users, groups, Conditional Access, access reviews, entitlement management, and lifecycle workflows are not modeled as standard ARM resources.
- For those tasks, use Portal, Microsoft Graph PowerShell, or Azure CLI with `az rest`.
