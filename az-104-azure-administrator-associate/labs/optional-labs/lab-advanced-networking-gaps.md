# Day 8 - Lab 3: Advanced Networking Gaps

## Objective
Cover remaining networking objectives: effective rules/ASG concepts, Bastion, service endpoints/private endpoints, Azure DNS, and UDR interpretation.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Network security groups** and review effective security rules for one VM NIC/subnet.
2. Open **Application security groups** and create one minimal ASG (or review existing) for rule targeting concept.
3. Open **Bastion** and review deployment prerequisites and connection model.
4. Open storage or another PaaS resource networking settings and compare service endpoint vs private endpoint flow.
5. Open **DNS zones** (public/private) and create one minimal zone/record if allowed.
6. Open route tables and confirm UDR association and route intent.
7. Note down one troubleshooting example using these controls.

## Azure CLI Helper
```bash
RG=rg-az104
az network nsg list --resource-group $RG --output table
az network asg list --resource-group $RG --output table
az network bastion list --resource-group $RG --output table
az network private-endpoint list --resource-group $RG --output table
az network route-table list --resource-group $RG --output table
az network dns zone list --resource-group $RG --output table
```

## Validation
- At least one advanced network control is configured or validated.
- Endpoint model differences (service vs private endpoint) are documented.
- DNS/UDR troubleshooting intent is captured.

## Complete Cleanup
- Remove temporary DNS/ASG/endpoint resources if they were created only for this lab.
