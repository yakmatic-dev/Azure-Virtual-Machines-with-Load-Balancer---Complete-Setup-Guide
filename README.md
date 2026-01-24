# Azure-Virtual-Machines-with-Load-Balancer---Complete-Setup-Guide

# Azure Virtual Machines with Load Balancer - Complete Setup Guide

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Configuration Details](#configuration-details)
- [Testing and Validation](#testing-and-validation)
- [Troubleshooting](#troubleshooting)
- [Cost Optimization](#cost-optimization)
- [Security Best Practices](#security-best-practices)
- [Maintenance and Monitoring](#maintenance-and-monitoring)
- [Clean Up Resources](#clean-up-resources)

---

## Overview

This guide walks you through creating a highly available web application infrastructure on Microsoft Azure using:
- Multiple Virtual Machines (VMs)
- Azure Load Balancer for traffic distribution
- Virtual Network for secure communication
- Health probes for automatic failover

**Use Cases:**
- Web applications requiring high availability
- APIs that need to handle variable traffic loads
- Services that require zero-downtime deployments
- Applications needing geographic redundancy

---

## Architecture

```
Internet
    |
    v
[Azure Load Balancer]
    |
    +-- Frontend IP (Public)
    |
    +-- Load Balancing Rule (Port 80)
    |
    +-- Health Probe (HTTP Check)
    |
    v
[Backend Pool]
    |
    +-- VM1 (Web Server)
    |
    +-- VM2 (Web Server)
    |
[Virtual Network]
    |
    +-- Subnet (10.0.0.0/24)
```

**Traffic Flow:**
1. User requests hit the Load Balancer's public IP
2. Load Balancer checks VM health via health probes
3. Traffic is distributed to healthy VMs using round-robin or hash-based algorithm
4. VMs process requests and return responses
5. Load Balancer forwards responses back to users

---

## Prerequisites

### Required Access
- Azure subscription with Contributor or Owner role
- Permissions to create resources in a resource group

### Required Knowledge
- Basic understanding of networking concepts (IP addresses, ports, protocols)
- Familiarity with Azure Portal navigation
- Basic Linux/Windows server administration

### Tools Needed
- Web browser to access Azure Portal
- SSH client (for Linux VMs) - PuTTY, Terminal, or Windows Terminal
- RDP client (for Windows VMs) - Remote Desktop Connection

### Estimated Costs
- 2 x B2s VMs: ~$30-40/month each
- Load Balancer (Standard): ~$20-25/month
- Public IP: ~$3-4/month
- Network egress: Variable based on traffic
- **Total: Approximately $85-110/month**

---

## Step-by-Step Setup

### Step 1: Create a Resource Group

A resource group is a container that holds related Azure resources.

1. Sign in to [Azure Portal](https://portal.azure.com)
2. Click **"Create a resource"** or search for **"Resource groups"**
3. Click **"+ Create"**
4. Fill in the details:
   - **Subscription:** Select your Azure subscription
   - **Resource group name:** `rg-loadbalancer-demo`
   - **Region:** `East US` (or your preferred region)
5. Click **"Review + Create"**
6. Click **"Create"**

**Best Practice:** Use consistent naming conventions. Example: `rg-[project]-[environment]-[region]`

<img width="1654" height="883" alt="image" src="https://github.com/user-attachments/assets/65912539-04b2-4976-8785-86d2e7a19dbe" />


---

### Step 2: Create a Virtual Network

1. Search for **"Virtual networks"** in the top search bar
2. Click **"+ Create"**
3. **Basics tab:**
   - **Subscription:** Your subscription
   - **Resource group:** `rg-loadbalancer-demo`
   - **Name:** `vnet-loadbalancer`
   - **Region:** Same as resource group (e.g., East US)
4. **IP Addresses tab:**
   - **IPv4 address space:** `10.31.0.0/24`
   - Click **"+ Add subnet"**
     - **Subnet name:** `subnet-vms`
     - **Subnet address range:** `10.31.1.0/24`
   - Click **"Add"**
5. **Security tab:** (optional, can configure later)
   - Leave defaults for now
6. Click **"Review + Create"**
7. Click **"Create"**

**Network Design Notes:**
- /16 network gives you 65,536 IP addresses
- /24 subnet gives you 256 IP addresses (Azure reserves 5)
- Plan for future growth when sizing subnets

---
<img width="1415" height="611" alt="image" src="https://github.com/user-attachments/assets/31585b9e-1f2b-49c2-aafa-f9ee4e8b1989" />

### Step 3: Create an Availability Set (Recommended)

Availability Sets ensure VMs are distributed across multiple fault domains and update domains.

1. Search for **"Availability sets"**
2. Click **"+ Create"**
3. Fill in:
   - **Subscription:** Your subscription
   - **Resource group:** `rg-loadbalancer-demo`
   - **Name:** `avset-webservers`
   - **Region:** Same region
   - **Fault domains:** 2
   - **Update domains:** 5
4. Click **"Review + Create"** then **"Create"**

**Why use Availability Sets?**
- Protects against hardware failures
- Ensures VMs aren't updated simultaneously
- Provides 99.95% SLA (vs 99.9% for single VM)

---
<img width="1235" height="618" alt="image" src="https://github.com/user-attachments/assets/b836045b-5212-401b-84fe-9f9eb698cdba" />

### Step 4: Create Virtual Machine 1

1. Search for **"Virtual machines"**
2. Click **"+ Create"** > **"Azure virtual machine"**

#### Basics Tab:
- **Subscription:** Your subscription
- **Resource group:** `rg-loadbalancer-demo`
- **Virtual machine name:** `vm-web-01`
- **Region:** Same region
- **Availability options:** Availability set
- **Availability set:** `avset-webservers`
- **Security type:** Standard
- **Image:** Ubuntu Server 22.04 LTS - Gen2 (or your preference)
- **Size:** Standard_B2s (2 vCPUs, 4 GB RAM)
- **Authentication type:** SSH public key (or Password)
- **Username:** `azureuser`
- **SSH public key source:** Generate new key pair
- **Key pair name:** `vm-web-keypair`

#### Disks Tab:
- **OS disk type:** Standard SSD (or Premium for production)
- Leave other defaults

#### Networking Tab:
- **Virtual network:** `vnet-loadbalancer`
- **Subnet:** `subnet-vms (10.0.1.0/24)`
- **Public IP:** Create new > Name: `pip-vm-web-01`
- **NIC network security group:** Basic
- **Public inbound ports:** Allow selected ports
- **Select inbound ports:** SSH (22), HTTP (80)

#### Management Tab:
- **Enable auto-shutdown:** Optional (saves costs in dev/test)
- Leave other defaults

#### Monitoring Tab:
- **Boot diagnostics:** Enable with managed storage account
- Leave other defaults

3. Click **"Review + Create"**
4. Click **"Create"**
5. **Download the private key** when prompted and save it securely

**Wait for deployment to complete** (2-3 minutes)

<img width="1237" height="625" alt="image" src="https://github.com/user-attachments/assets/c0ae3858-6c86-4e75-a9bd-304247d9cfaf" />

---

### Step 5: Create Virtual Machine 2

Repeat Step 4 with these changes:
- **Virtual machine name:** `vm-web-02`
- **Availability set:** `avset-webservers` (same as VM1)
- **SSH public key source:** Use existing key stored in Azure
- **Stored Keys:** Select `vm-web-keypair`
- **Public IP:** Create new > Name: `pip-vm-web-02`

**All other settings remain the same**
<img width="1191" height="622" alt="image" src="https://github.com/user-attachments/assets/4816c23f-500d-41b2-aaec-6f24db40945d" />

---
Step 6: Deploy Your Application Using a CI/CD Pipeline
Instead of manually installing and configuring the web server on each VM, we will automate application deployment using a CI/CD pipeline. This ensures consistency, faster deployments, and reduced human error.
High-Level Flow

Developer pushes code to the repository
CI/CD pipeline is triggered
Pipeline connects to VM1 and VM2
Application and web server configuration are deployed automatically
Application becomes available behind the Load Balancer

Prerequisites

VM1 and VM2 are reachable over SSH
SSH private key is securely stored in the CI/CD tool (e.g., GitHub Actions Secrets, Azure DevOps Library)
Load Balancer is already configured
Source code repository is set up

Example: GitHub Actions CI/CD Pipeline
Create a file named .github/workflows/deploy.yml in your repository:
yamlname: Deploy Application to Azure VMs

name: Build and Deploy App to Azure VMs

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  NODE_VERSION: '24'
  APP_NAME: 'ts-app'
  APP_PORT: 4200
  REPO_PATH: 'app'

jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        vm:
          - name: VM1
            host_secret: VM1_PUBLIC_IP
            username_secret: VM1_USERNAME
            key_secret: VM1_SSH_PRIVATE_KEY
          - name: VM2
            host_secret: VM2_PUBLIC_IP
            username_secret: VM2_USERNAME
            key_secret: VM2_SSH_PRIVATE_KEY
      fail-fast: false
      max-parallel: 2
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Copy code to ${{ matrix.vm.name }}
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets[matrix.vm.host_secret] }}
          username: ${{ secrets[matrix.vm.username_secret] }}
          key: ${{ secrets[matrix.vm.key_secret] }}
          source: "."
          target: ${{ env.REPO_PATH }}
          rm: true
          overwrite: true

      - name: Build and Deploy on ${{ matrix.vm.name }}
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets[matrix.vm.host_secret] }}
          username: ${{ secrets[matrix.vm.username_secret] }}
          key: ${{ secrets[matrix.vm.key_secret] }}
          command_timeout: 20m
          script: |
            set -e
            
            echo "=========================================="
            echo "🚀 Starting deployment on ${{ matrix.vm.name }}"
            echo "=========================================="
            
            # Navigate to app directory
            cd ~/${{ env.REPO_PATH }}
            
            echo "📁 Files copied successfully"
            ls -la
            pwd
          
            
            # Install Node.js dependencies
            echo ""
            echo "📥 Installing dependencies..."
            npm install
            npm run build
            
            # Stop existing PM2 process
            echo ""
            echo "🛑 Stopping existing application..."
            pm2 stop ${{ env.APP_NAME }} 2>/dev/null || true
            pm2 delete ${{ env.APP_NAME }} 2>/dev/null || true
            
            # Start application with PM2
            echo ""
            echo "▶️  Starting Angular application..."
            pm2 start npm \
              --name ${{ env.APP_NAME }} \
              --interpreter none \
              -- run start -- --host 0.0.0.0
            
            # Save PM2 configuration
            pm2 save --force
            
            # Setup PM2 to start on system boot (only runs once)
            pm2 startup systemd -u ${{ secrets[matrix.vm.username_secret] }} --hp /home/${{ secrets[matrix.vm.username_secret] }} 2>/dev/null || true
            
            # Wait for application to start
            echo ""
            echo "⏳ Waiting for Angular to compile..."
            sleep 20
            
            # Check application status
            echo ""
            echo "=========================================="
            echo "📊 Application Status"
            echo "=========================================="
            pm2 status ${{ env.APP_NAME }}
            
            # Show recent logs
            echo ""
            echo "=========================================="
            echo "📝 Recent Logs (last 25 lines)"
            echo "=========================================="
            pm2 logs ${{ env.APP_NAME }} --lines 25 --nostream
            
            # Verify application is running
            if pm2 status ${{ env.APP_NAME }} | grep -q "online"; then
              echo ""
              echo "=========================================="
              echo "✅ Deployment successful on ${{ matrix.vm.name }}"
              echo "=========================================="
            else
              echo ""
              echo "=========================================="
              echo "❌ Deployment failed on ${{ matrix.vm.name }}"
              echo "=========================================="
              exit 1
            fi

      - name: Health check on ${{ matrix.vm.name }}
        if: success()
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets[matrix.vm.host_secret] }}
          username: ${{ secrets[matrix.vm.username_secret] }}
          key: ${{ secrets[matrix.vm.key_secret] }}
          script: |
            # Wait for application to be fully ready
            sleep 5
            
            # Test if Angular dev server is responding
            if curl -f -s --max-time 10 http://localhost:${{ env.APP_PORT }} > /dev/null 2>&1; then
              echo "✅ Health check passed - Application is responding on port ${{ env.APP_PORT }}"
            else
              echo "⚠️  Application not responding on port ${{ env.APP_PORT }} yet (may still be compiling)"
            fi
            
            # Show final PM2 status
            pm2 status ${{ env.APP_NAME }}

  notify:
    runs-on: ubuntu-latest
    needs: deploy
    if: always()
    steps:
      - name: Deployment Summary
        run: |
          echo "=========================================="
          if [ "${{ needs.deploy.result }}" == "success" ]; then
            echo "✅ Deployment completed successfully on all VMs"
            echo "Commit: ${{ github.sha }}"
            echo "Branch: ${{ github.ref_name }}"
            echo "=========================================="
          else
            echo "❌ Deployment failed or partially completed"
            echo "=========================================="
            exit 1
          fi
Setting Up GitHub Secrets

Navigate to your repository on GitHub
Go to Settings > Secrets and variables > Actions
Click New repository secret
Add the following secrets:

VM1_PUBLIC_IP: Public IP address of VM1
VM2_PUBLIC_IP: Public IP address of VM2
SSH_PRIVATE_KEY: Your SSH private key content
### Step 6: Install Web Server on Both VMs

#### Connect to VM1:

**On Linux/Mac:**
```bash
chmod 400 ~/Downloads/vm-web-keypair.pem
ssh -i ~/Downloads/vm-web-keypair.pem azureuser@<VM1-PUBLIC-IP>
```

**On Windows (PowerShell):**
```powershell
ssh -i C:\Users\YourName\Downloads\vm-web-keypair.pem azureuser@<VM1-PUBLIC-IP>
```

#### Verify the app is running on both VM:

<img width="1560" height="680" alt="image" src="https://github.com/user-attachments/assets/cfc3d6e9-1789-41f5-ba1c-7de4763a9be7" />


#### Repeat for VM2:

# Connect to VM2
ssh -i ~/Downloads/vm-web-keypair.pem azureuser@<VM2-PUBLIC-IP>

<img width="988" height="141" alt="image" src="https://github.com/user-attachments/assets/42d0f1dd-117f-4602-a72b-8376ccedd236" />




#### Test Individual VMs:
Open a browser and navigate to:
- `http://<VM1-PUBLIC-IP>:port` - 
- `http://<VM2-PUBLIC-IP>` :port-" see the app running"

---
<img width="1909" height="911" alt="image" src="https://github.com/user-attachments/assets/886e038f-cd36-4fa2-8348-2bf678967b26" />

### Step 7: Create Azure Load Balancer

1. Search for **"Load balancers"**
2. Click **"+ Create"**

#### Basics Tab:
- **Subscription:** Your subscription
- **Resource group:** `rg-loadbalancer-demo`
- **Name:** `lb-web-frontend`
- **Region:** Same region
- **SKU:** Standard
- **Type:** Public
- **Tier:** Regional

#### Frontend IP Configuration Tab:
1. Click **"+ Add a frontend IP configuration"**
2. Fill in:
   - **Name:** `frontend-public-ip`
   - **IP version:** IPv4
   - **IP type:** IP address
   - **Public IP address:** Create new
     - **Name:** `pip-loadbalancer`
     - **SKU:** Standard
     - **Assignment:** Static
     - **Routing preference:** Microsoft network (recommended)
   - **Availability zone:** Zone-redundant
3. Click **"Add"**

#### Backend Pools Tab:
1. Click **"+ Add a backend pool"**
2. Fill in:
   - **Name:** `backend-pool-webservers`
   - **Virtual network:** `vnet-loadbalancer`
   - **Backend Pool Configuration:** NIC
   - **IP Version:** IPv4
3. Click **"+ Add"** under Virtual machines
4. Select both VMs:
   - Check `vm-web-01`
   - Check `vm-web-02`
5. Click **"Add"**
6. Click **"Save"**

3. Click **"Review + Create"**
4. Click **"Create"**

**Wait for deployment** (1-2 minutes)

---
<img width="1223" height="625" alt="image" src="https://github.com/user-attachments/assets/d94e9fe6-6fdf-4a00-8c50-331e868c74fc" />


### Step 8: Configure Health Probe

1. Navigate to your Load Balancer resource: `lb-web-frontend`
2. Under **Settings**, click **"Health probes"**
3. Click **"+ Add"**
4. Fill in:
   - **Name:** `health-probe-http`
   - **Protocol:** HTTP
   - **Port:** 4200
   - **Path:** `/`
   - **Interval:** 5 seconds
   - **Unhealthy threshold:** 2 consecutive failures
5. Click **"Save"**

**How it works:**
- Load balancer sends HTTP GET request to `http://<VM-IP>/` every 5 seconds
- If VM doesn't respond or returns error code, it's marked unhealthy after 2 failures
- Unhealthy VMs are removed from rotation automatically

---
<img width="1692" height="701" alt="image" src="https://github.com/user-attachments/assets/c82b6487-c441-412a-b5b4-cf523c8954d3" />


### Step 9: Create Load Balancing Rule

1. Still in the Load Balancer, under **Settings**, click **"Load balancing rules"**
2. Click **"+ Add"**
3. Fill in:
   - **Name:** `rule-http-traffic`
   - **IP Version:** IPv4
   - **Frontend IP address:** `frontend-public-ip`
   - **Backend pool:** `backend-pool-webservers`
   - **Protocol:** TCP
   - **Port:** 80
   - **Backend port:** 4200
   - **Health probe:** `health-probe-http`
   - **Session persistence:** None
   - **Idle timeout (minutes):** 4
   - **TCP reset:** Enabled
   - **Floating IP:** Disabled
   - **Outbound source network address translation (SNAT):** Use default
4. Click **"Save"**

**Session Persistence Options:**
- **None:** Requests distributed to any healthy VM (recommended for stateless apps)
- **Client IP:** Same client goes to same VM
- **Client IP and protocol:** Same client+protocol goes to same VM

---
<img width="1542" height="895" alt="image" src="https://github.com/user-attachments/assets/bdfb510c-2340-4f2f-b21f-c22754553da3" />

<img width="1916" height="908" alt="image" src="https://github.com/user-attachments/assets/d02649a2-b9e0-4645-87da-be3075f6ee9d" />


### Step 10: Update Network Security Groups

Remove direct access to VMs and only allow traffic through Load Balancer.

#### Update VM1 NSG:
1. Go to `vm-web-01` resource
2. Click **"Networking"** under Settings
3. Click on the NSG name (e.g., `vm-web-01-nsg`)
4. Under **Settings**, click **"Inbound security rules"**
5. Find the  rule (port 4200)
6. Click the **"..."** menu and **"Delete"** (we'll access via LB only)
7. Keep SSH rule for management access

#### Repeat for VM2

**Security Note:** In production, use Azure Bastion for SSH access and remove public IPs from VMs entirely.

---

## Configuration Details

### Load Balancer Settings Explained

| Setting | Value | Purpose |
|---------|-------|---------|
| SKU | Standard | Production-ready with availability zones, more features |
| Type | Public | Internet-facing (use Internal for private apps) |
| Session Persistence | None | Stateless load balancing for better distribution |
| Health Probe Interval | 5 seconds | Quick detection of unhealthy instances |
| Unhealthy Threshold | 2 failures | Balance between sensitivity and stability |

### Virtual Network Configuration

| Component | CIDR | Available IPs |
|-----------|------|---------------|
| VNet | 10.0.0.0/16 | 65,536 |
| Subnet | 10.0.1.0/24 | 251 usable (256 - 5 reserved by Azure) |

### VM Specifications

| Component | Specification | Notes |
|-----------|---------------|-------|
| Size | Standard_B2s | 2 vCPUs, 4 GB RAM - good for dev/test |
| OS Disk | Standard SSD | 30 GB - sufficient for OS and web server |
| OS | Ubuntu 22.04 LTS | Long-term support, regular security updates |
| Availability | Availability Set | 99.95% SLA |

---

## Testing and Validation

### Test 1: Basic Load Balancing

1. Go to Load Balancer > **Overview**
2. Copy the **Frontend IP address**
3. Open browser and navigate to: `http://<LOAD-BALANCER-IP>`
4. Refresh the page multiple times
5. You should see responses alternating between VM1 and VM2

**Expected Result:**
```
First load:  Hello from VM1 - vm-web-01
Second load: Hello from VM2 - vm-web-02
Third load:  Hello from VM1 - vm-web-01
...
```
<img width="1916" height="908" alt="image" src="https://github.com/user-attachments/assets/95eaaa93-7f8c-4edb-b253-446d30509651" />

### Test 2: Health Probe Functionality

1. SSH into VM1
2. Stop app:
  
   ```
3. Wait 10-15 seconds (2 health probe failures)
4. Refresh the Load Balancer IP in browser
5. You should ONLY see responses from VM2

6. Start app again:
  
7. Wait 10 seconds
8. VM1 should be back in rotation

**Check Health Status:**
- Go to Load Balancer > **Backend pools**
- Click `backend-pool-webservers`
- View the health status of each VM

### Test 3: Load Testing (Optional)

Use `curl` in a loop to test distribution:

```bash
for i in {1..20}; do
  curl http://<LOAD-BALANCER-IP>
  echo ""
done
```

Or use Apache Bench:
```bash
ab -n 1000 -c 10 http://<LOAD-BALANCER-IP>/
```

### Test 4: Verify Metrics

1. Go to Load Balancer > **Metrics**
2. Add these metrics:
   - **Data Path Availability** (should be 100%)
   - **Health Probe Status** (should show 100% for healthy VMs)
   - **Byte Count**
   - **Packet Count**

---

## Troubleshooting

### Issue: Can't Access Load Balancer IP

**Possible Causes:**
1. Health probes failing
2. Backend pool empty
3. Load balancing rule not configured

**Solutions:**
```bash
# Check if VMs are responding locally
curl localhost:port

# Check app status
pm2 status
pm2 logs

# Restart app
pm2 restart

# Check firewall
sudo ufw status
```

**In Azure Portal:**
- Verify health probe shows VMs as healthy
- Check NSG rules allow port 4200
- Verify backend pool has VMs added

### Issue: Always Seeing Same VM

**Cause:** Session persistence enabled or browser caching

**Solution:**
- Check Load Balancer rule > Session persistence = None
- Use different browsers or incognito mode
- Clear browser cache
- Use `curl` instead: `curl -s http://<LB-IP>`

### Issue: Intermittent Failures

**Possible Causes:**
1. One VM is unhealthy
2. Health probe misconfigured
3. Application errors

**Diagnostics:**
```bash
# Check app error logs

pm2 logs


# Test local connectivity
curl -v http://localhost
```

**In Azure:**
- Check Metrics > Health Probe Status
- Review Activity Log for any changes
- Check VM metrics (CPU, memory, disk)

### Issue: High Latency

**Solutions:**
- Upgrade VM size (more vCPUs/RAM)
- Use Premium SSD for OS disk
- Ensure VMs and LB in same region
- Check network performance metrics
- Consider Azure Application Gateway for Layer 7 optimization

---

## Cost Optimization

### Development/Testing

1. **Use B-series VMs:** Burstable performance, lower cost
2. **Auto-shutdown:** Enable on VMs during non-business hours
3. **Deallocate when not in use:**
   ```bash
   az vm deallocate --resource-group rg-loadbalancer-demo --name vm-web-01
   az vm deallocate --resource-group rg-loadbalancer-demo --name vm-web-02
   ```
4. **Use Spot VMs:** Up to 90% discount (for fault-tolerant workloads)

### Production

1. **Reserved Instances:** 1-year or 3-year commitment saves 40-60%
2. **Azure Hybrid Benefit:** Use existing Windows licenses
3. **Right-size VMs:** Monitor and adjust based on actual usage
4. **Use Standard Load Balancer efficiently:** Consolidate rules
5. **Monitor bandwidth:** Egress charges can add up

### Cost Monitoring

Set up budget alerts:
1. Go to **Cost Management + Billing**
2. Click **Budgets**
3. Create budget for resource group
4. Set alerts at 50%, 75%, 90%, 100%

**Estimated Monthly Costs:**
- 2x B2s VMs (730 hrs): ~$60-80
- Standard Load Balancer: ~$20-25
- Standard Public IP: ~$3-4
- Bandwidth (first 100 GB free): Variable
- **Total: $85-110/month**

---

## Security Best Practices

### Network Security

1. **Remove Public IPs from VMs:**
   - Only Load Balancer should be internet-facing
   - Use Azure Bastion for secure VM access

2. **Configure NSG Rules Properly:**
   ```
   Priority  Name           Port  Source              Destination
   100       AllowLB        80    AzureLoadBalancer   VirtualNetwork
   110       AllowBastion   22    AzureBastionSubnet  VirtualNetwork
   200       DenyAll        *     *                   *
   ```

3. **Enable DDoS Protection:**
   - Standard Load Balancer includes basic DDoS protection
   - Consider DDoS Protection Standard for critical apps

### Application Security

1. **Keep Systems Updated:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Enable HTTPS:**
   - Install SSL certificates
   - Configure Nginx for SSL
   - Or use Application Gateway for SSL termination

3. **Implement Web Application Firewall:**
   - Use Azure Application Gateway with WAF
   - Protects against OWASP Top 10 vulnerabilities

### Multi-Region Deployment

For global applications:
1. Deploy same architecture in multiple regions
2. Use Azure Traffic Manager for global load balancing
3. Configure geo-replication for data
4. Implement CDN for static content

### Monitoring with Application Insights

1. Install Application Insights agent on VMs
2. Configure custom telemetry
3. Create availability tests
4. Set up smart detection alerts

---

## Clean Up Resources

### Option 1: Delete Resource Group (Deletes Everything)

```bash
az group delete --name rg-loadbalancer-demo --yes --no-wait
```

**Or in Portal:**
1. Go to Resource Groups
2. Click `rg-loadbalancer-demo`
3. Click **"Delete resource group"**
4. Type the resource group name to confirm
5. Click **"Delete"**

### Option 2: Delete Individual Resources

If you want to keep some resources:

1. Delete Load Balancer
2. Delete VMs (this also deletes disks, NICs, public IPs)
3. Delete Availability Set
4. Delete Virtual Network
5. Finally delete Resource Group

### Verify Deletion

1. Go to **Cost Management + Billing**
2. Check **Cost Analysis**
3. Verify no charges from deleted resources
4. May take 24-48 hours to reflect

---

## Additional Resources

### Official Documentation
- [Azure Load Balancer Documentation](https://docs.microsoft.com/azure/load-balancer/)
- [Azure Virtual Machines Documentation](https://docs.microsoft.com/azure/virtual-machines/)
- [Azure Networking Documentation](https://docs.microsoft.com/azure/networking/)

### Pricing Calculators
- [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)
- [Total Cost of Ownership Calculator](https://azure.microsoft.com/pricing/tco/)

### Learning Resources
- [Microsoft Learn - Azure Fundamentals](https://learn.microsoft.com/training/paths/azure-fundamentals/)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)

### Support
- [Azure Support Plans](https://azure.microsoft.com/support/plans/)
- [Azure Community Forums](https://docs.microsoft.com/answers/products/azure)
- [Stack Overflow - Azure Tag](https://stackoverflow.com/questions/tagged/azure)

---
## Project by 

Yakub Ilyas
yakubiliyas12@gmail.com

