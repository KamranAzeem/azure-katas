# Day 2 - Lab 3: Connectivity and Load Balancing Starter

## Context (Why this lab matters)
- AZ-104 requires understanding frontend exposure, backend health, and traffic distribution.
- This lab builds a practical load balancer troubleshooting baseline.
- You should finish with details that probe/rule/backend wiring behaves as expected.

## Objective
Deploy a basic public load balancer with two Linux backend VMs, health probe, and load-balancing rule, then validate backend readiness.

## Scenario
You need to expose a simple web workload through a resilient frontend endpoint.
You configure LB components and verify backend health and request distribution.

## Direct Portal GUI Steps (Primary)
1. Ensure a backend subnet exists (for example from Day 2 Lab 2: `vnet-az104-d2` / `snet-app`).
2. Create two Linux backend VMs in the backend subnet:
   - `vm-az104-d2-web-01`
   - `vm-az104-d2-web-02`
   - Apply settings from **Portal VM Defaults (Cost-Optimized)**.
3. On both VMs, install a basic web service (for example nginx) and verify TCP `80` is listening.
    - SSH to each VM:
       - `ssh azureuser@<publicIP>`
    - Install web server:
       - `sudo apt-get update && sudo apt-get -y install nginx`
    - Exit the VM and repeat for the second backend VM.
4. Ensure both backend VM NSGs allow inbound HTTP (`TCP 80`) so LB traffic can reach nginx.
   - Add inbound rule `allow-http` on both VM NSGs (for example `vm-az104-d2-web-01-nsg` and `vm-az104-d2-web-02-nsg`).
5. Open Azure Portal -> **Load balancers** -> **Create**.
6. In **Basics**, choose:
   - **SKU**: `Standard load balancer`
   - **Type**: `Public`
7. Use `rg-az104`, create/attach public IP, and name LB `lb-az104-d2`.
8. In LB settings, configure frontend IP and create backend pool `web-servers`; add both backend VM NICs to the pool.
9. Create health probe `tcp-80-probe` (protocol `TCP`, port `80`, interval `5s`, unhealthy threshold `2`).
10. Create load-balancing rule `tcp-80-rule` (frontend `80` -> backend `80`) and bind it to backend pool `web-servers` and probe `tcp-80-probe`.
11. Verify load balancing behavior from your home/work computer:
    - Terminal A: SSH to `vm-az104-d2-web-01` and run `sudo tail -f /var/log/nginx/access.log`
    - Terminal B: SSH to `vm-az104-d2-web-02` and run `sudo tail -f /var/log/nginx/access.log`
    - Terminal C: run repeated requests to LB public IP:
       - `for i in {1..20}; do curl -s http://<lb-public-ip>; sleep 1; done`
    - Confirm both backend VM logs show incoming requests.
12. Review LB **Frontend IP configuration**, **Backend pools**, **Health probes**, and **Load balancing rules** blades.
13. Note down backend VM names, probe config, rule config, and LB public IP.

### Portal VM Defaults (Cost-Optimized)
- **Security tab**
   - Security type: `Standard`
   - Availability options: `No infrastructure redundancy required`
   - Image: `Ubuntu Server (latest available)`
   - Size: choose the lowest-cost size available in the region
   - Inbound ports: allow `SSH (22)` only
- **Disks tab**
   - OS disk type: `Standard SSD`
   - Delete with VM: `Enabled`
- **Networking tab**
   - Virtual network: pick an existing VNet for the lab context
   - Subnet: use the backend subnet for this lab
   - Public IP: `Create new`
- **Monitoring tab**
   - Diagnostics / Boot diagnostics: `Disabled`

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --output table
az network lb show --resource-group $RG --name lb-az104-d2 --query "{Name:name,Location:location}" --output table
az network lb rule list --resource-group $RG --lb-name lb-az104-d2 --output table
az network lb probe list --resource-group $RG --lb-name lb-az104-d2 --output table
az network lb address-pool show --resource-group $RG --lb-name lb-az104-d2 --name web-servers --query "{Name:name,BackendCount:length(backendIPConfigurations)}" --output table
az vm list -d --resource-group $RG --query "[?name=='vm-az104-d2-web-01' || name=='vm-az104-d2-web-02'].{Name:name,PrivateIP:privateIps,PublicIP:publicIps,Power:powerState}" --output table
```

## Optional Deep-Dive Exercise (Proof of LB Impact)
Use this only if you want a clear break/fix demonstration.

### Create
1. Keep both backend VMs running nginx and keep LB rule/probe as configured.
2. Confirm baseline works:
   - `for i in {1..10}; do curl -s http://<lb-public-ip>; sleep 1; done`

### Validate
1. Introduce a controlled break on one backend VM NSG (for example remove or deny `allow-http` on `vm-az104-d2-web-01-nsg`).
2. Re-run LB traffic test:
   - `for i in {1..20}; do curl -s http://<lb-public-ip>; sleep 1; done`
3. Observe probe/backend behavior in Portal (**Load balancer -> Backend pools / Health probes**) and backend access logs.
   - Expected: degraded backend distribution or unhealthy backend signal.
4. Restore `allow-http` on the affected backend NSG and re-test.
   - Expected: backend health/distribution returns to normal.

### Cleanup
1. Remove only temporary break/fix changes made for this optional test.
2. Keep baseline lab resources unchanged unless you run full lab cleanup.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day2/scenario03/main.json`

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
VNET=vnet-az104-d2
SNET=snet-app
LB=lb-az104-d2
PIP=publicip-az104-d2-lb
FE=fe-01
BE=web-servers
PROBE=tcp-80-probe
RULE=tcp-80-rule
VM1=vm-az104-d2-web-01
VM2=vm-az104-d2-web-02
ADMIN_USER=azureuser
PUBKEY=$(cat ~/.ssh/id_rsa.pub)
az account show --output table
az group show --name $RG --output table
az vm create --resource-group $RG --location $LOC --name $VM1 --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $SNET --public-ip-sku Standard
az vm create --resource-group $RG --location $LOC --name $VM2 --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $SNET --public-ip-sku Standard
az network nsg rule create --resource-group $RG --nsg-name ${VM1}-nsg --name allow-http --priority 310 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80
az network nsg rule create --resource-group $RG --nsg-name ${VM2}-nsg --name allow-http --priority 310 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80
IP1=$(az vm list-ip-addresses --resource-group $RG --name $VM1 --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)
IP2=$(az vm list-ip-addresses --resource-group $RG --name $VM2 --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)
ssh -o StrictHostKeyChecking=no $ADMIN_USER@$IP1 "sudo apt-get update -y && sudo apt-get install -y nginx && echo '$VM1' | sudo tee /var/www/html/index.nginx-debian.html && sudo systemctl enable --now nginx"
ssh -o StrictHostKeyChecking=no $ADMIN_USER@$IP2 "sudo apt-get update -y && sudo apt-get install -y nginx && echo '$VM2' | sudo tee /var/www/html/index.nginx-debian.html && sudo systemctl enable --now nginx"
az network lb create --resource-group $RG --location $LOC --name $LB --public-ip-address $PIP --frontend-ip-name $FE --backend-pool-name $BE
NIC1=$(az vm show --resource-group $RG --name $VM1 --query "networkProfile.networkInterfaces[0].id" -o tsv)
NIC2=$(az vm show --resource-group $RG --name $VM2 --query "networkProfile.networkInterfaces[0].id" -o tsv)
NIC1_NAME=$(basename $NIC1)
NIC2_NAME=$(basename $NIC2)
az network nic ip-config address-pool add --resource-group $RG --nic-name $NIC1_NAME --ip-config-name ipconfig1 --address-pool $BE --lb-name $LB
az network nic ip-config address-pool add --resource-group $RG --nic-name $NIC2_NAME --ip-config-name ipconfig1 --address-pool $BE --lb-name $LB
az network lb probe create --resource-group $RG --lb-name $LB --name $PROBE --protocol Tcp --port 80 --interval 5 --threshold 2
az network lb rule create --resource-group $RG --lb-name $LB --name $RULE --protocol Tcp --frontend-port 80 --backend-port 80 --frontend-ip-name $FE --backend-pool-name $BE --probe-name $PROBE
az network lb show --resource-group $RG --name $LB --output table
az network lb address-pool show --resource-group $RG --lb-name $LB --name $BE --query "{BackendCount:length(backendIPConfigurations)}" --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && VNET=vnet-az104-d2 && SNET=snet-app && LB=lb-az104-d2 && PIP=publicip-az104-d2-lb && FE=fe-01 && BE=web-servers && PROBE=tcp-80-probe && RULE=tcp-80-rule && VM1=vm-az104-d2-web-01 && VM2=vm-az104-d2-web-02 && ADMIN_USER=azureuser && PUBKEY=$(cat ~/.ssh/id_rsa.pub) && az account show --output table && az group create --name $RG --location $LOC --output table && az vm create --resource-group $RG --location $LOC --name $VM1 --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $SNET --public-ip-sku Standard && az vm create --resource-group $RG --location $LOC --name $VM2 --image Ubuntu2404 --size Standard_B2ls_v2 --admin-username $ADMIN_USER --authentication-type ssh --ssh-key-values "$PUBKEY" --vnet-name $VNET --subnet $SNET --public-ip-sku Standard && az network nsg rule create --resource-group $RG --nsg-name ${VM1}-nsg --name allow-http --priority 310 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80 && az network nsg rule create --resource-group $RG --nsg-name ${VM2}-nsg --name allow-http --priority 310 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80 && az network lb create --resource-group $RG --location $LOC --name $LB --public-ip-address $PIP --frontend-ip-name $FE --backend-pool-name $BE && az network lb probe create --resource-group $RG --lb-name $LB --name $PROBE --protocol Tcp --port 80 --interval 5 --threshold 2 && az network lb rule create --resource-group $RG --lb-name $LB --name $RULE --protocol Tcp --frontend-port 80 --backend-port 80 --frontend-ip-name $FE --backend-pool-name $BE --probe-name $PROBE && az network lb show --resource-group $RG --name $LB --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
2. Ensure backend network exists (`vnet-az104-d2` and target subnet).
3. Create two backend Linux VMs (`vm-az104-d2-web-01`, `vm-az104-d2-web-02`) in that subnet.
4. Install nginx on both VMs and confirm they listen on TCP `80`.
   - `ssh azureuser@<publicIP>`
   - `sudo apt-get update && sudo apt-get -y install nginx`
   - Repeat on both backend VMs.
5. Add NSG HTTP allow rule on both backend VM NSGs:
   - `az network nsg rule create --resource-group $RG --nsg-name vm-az104-d2-web-01-nsg --name allow-http --priority 310 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80`
   - `az network nsg rule create --resource-group $RG --nsg-name vm-az104-d2-web-02-nsg --name allow-http --priority 310 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 80`
6. Create LB resources:
   - Public IP: `publicip-az104-d2-lb`
   - Load balancer: `lb-az104-d2`
   - Frontend: `fe-01`
   - Backend pool: `web-servers`
   - Probe: `tcp-80-probe` (TCP `80`)
   - Rule: `tcp-80-rule` (frontend `80` to backend `80`)
7. Add both VM NICs to backend pool `web-servers`.
8. Verify load balancer and pool membership:
   - `az network lb show --resource-group $RG --name lb-az104-d2 --output table`
   - `az network lb probe list --resource-group $RG --lb-name lb-az104-d2 --output table`
   - `az network lb rule list --resource-group $RG --lb-name lb-az104-d2 --output table`
   - `az network lb address-pool show --resource-group $RG --lb-name lb-az104-d2 --name web-servers --query "{BackendCount:length(backendIPConfigurations)}" --output table`
9. Runtime validation from your home/work computer:
   - Terminal A: `ssh azureuser@<vm1-public-ip>` then `sudo tail -f /var/log/nginx/access.log`
   - Terminal B: `ssh azureuser@<vm2-public-ip>` then `sudo tail -f /var/log/nginx/access.log`
   - Terminal C: `for i in {1..20}; do curl -s http://<lb-public-ip>; sleep 1; done`
   - Confirm both backend access logs show activity.

## Validation
- Two backend Linux VMs exist and are running web service on TCP `80`.
- Backend VM NSGs allow inbound `TCP 80`.
- LB frontend, backend pool, probe, and rule exist.
- Backend pool contains both backend VM NICs.
- Accessing LB public IP on port `80` returns a web response.
- Repeated requests to LB public IP are visible in nginx access logs on both backend VMs.
- Optional deep-dive path: controlled break/fix on one backend NSG shows observable LB health/distribution impact and recovery.

## Complete Cleanup
- `az vm delete --resource-group rg-az104 --name vm-az104-d2-web-01 --yes`
- `az vm delete --resource-group rg-az104 --name vm-az104-d2-web-02 --yes`
- `az vm wait --resource-group rg-az104 --name vm-az104-d2-web-01 --deleted`
- `az vm wait --resource-group rg-az104 --name vm-az104-d2-web-02 --deleted`
- `az network lb delete --resource-group rg-az104 --name lb-az104-d2`
- `az network public-ip delete --resource-group rg-az104 --name publicip-az104-d2-lb`
- Post-check for leftovers:
   - `az resource list -g rg-az104 --query "[?contains(name,'vm-az104-d2-web') || name=='lb-az104-d2' || name=='publicip-az104-d2-lb'].{Type:type,Name:name}" -o table`
- If leftovers remain, remove VM-linked artifacts:
   - `az network nic list -g rg-az104 --query "[?contains(name,'vm-az104-d2-web') && virtualMachine==null].name" -o tsv | xargs -r -I{} az network nic delete -g rg-az104 -n {}`
   - `az network public-ip list -g rg-az104 --query "[?contains(name,'vm-az104-d2-web') && ipConfiguration==null].name" -o tsv | xargs -r -I{} az network public-ip delete -g rg-az104 -n {}`
   - `az network nsg list -g rg-az104 --query "[?contains(name,'vm-az104-d2-web') && length(networkInterfaces)==\`0\` && length(subnets)==\`0\`].name" -o tsv | xargs -r -I{} az network nsg delete -g rg-az104 -n {}`
- Final verification:
   - `az resource list -g rg-az104 --query "[?contains(name,'vm-az104-d2-web') || name=='lb-az104-d2' || name=='publicip-az104-d2-lb'].{Type:type,Name:name}" -o table`
