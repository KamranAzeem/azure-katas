# Day 8 - Lab 4: Monitor + Backup/DR Gaps

## Objective
Cover remaining monitor/recovery objectives: alert processing rules, Azure Monitor insights, Network Watcher/Connection Monitor, and Site Recovery concepts.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Monitor** -> **Alerts** and review/create one alert processing rule.
2. Open **Insights** for VM/storage/network and capture one practical metric/log interpretation.
3. Open **Network Watcher** and run one connection monitor or troubleshoot check.
4. Open backup/vault resources and review backup policy/reporting alerts.
5. Open **Recovery Services vault** and review Site Recovery setup flow and failover prerequisites.
6. Note down what is fully practiced vs concept-only due to environment constraints.

## Azure CLI Helper
```bash
RG=rg-az104
az monitor action-group list --resource-group $RG --output table
az monitor scheduled-query list --resource-group $RG --output table
az network watcher list --output table
az network watcher connection-monitor list --location northeurope --output table
az backup vault list --resource-group $RG --output table
```

## Validation
- Alert processing and monitor insights paths are demonstrated.
- Network Watcher/Connection Monitor output is captured.
- Backup/DR readiness notes include Site Recovery failover concepts.

## Complete Cleanup
- Remove temporary alert processing rules and monitor test artifacts created only for this lab.
