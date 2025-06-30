# Linux Commands for Azure-GCP VPN Project

## Project Setup Commands

### Authentication Setup
```bash
# Azure CLI login
az login
az account set --subscription "7d939770-deef-433d-9283-5bd5eb79aeaf"
az account show

# Google Cloud SDK login
gcloud auth login
gcloud config set project terraform-vpn-464313
gcloud auth application-default login
```

### Terraform Installation and Setup
```bash
# Install Terraform (Ubuntu/Debian)
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform

# Verify installation
terraform --version

# Initialize project
terraform init
```

## GCP API Management

### Enable Required APIs
```bash
# Enable Compute Engine API
gcloud services enable compute.googleapis.com --project=terraform-vpn-464313

# Enable additional required APIs
gcloud services enable \
  compute.googleapis.com \
  cloudresourcemanager.googleapis.com \
  iam.googleapis.com \
  --project=terraform-vpn-464313

# List enabled services
gcloud services list --enabled --project=terraform-vpn-464313

# Check specific API status
gcloud services list --enabled --filter="name:compute.googleapis.com" --project=terraform-vpn-464313
```

## Terraform Operations

### Basic Deployment Commands
```bash
# Plan deployment
terraform plan

# Apply configuration
terraform apply

# Apply with auto-approve (use with caution)
terraform apply -auto-approve

# Destroy infrastructure
terraform destroy
```

### State Management Commands
```bash
# List resources in state
terraform state list

# Show specific resource details
terraform state show module.azure_networking.azurerm_virtual_network_gateway.main

# Import existing resource
terraform import module.azure_networking.azurerm_subnet_route_table_association.main \
  /subscriptions/7d939770-deef-433d-9283-5bd5eb79aeaf/resourceGroups/rg-vpn-demo/providers/Microsoft.Network/virtualNetworks/vpn-demo-vnet-dev/subnets/vpn-demo-subnet-dev

# Remove resource from state
terraform state rm module.azure_networking.azurerm_subnet_route_table_association.main

# Refresh state
terraform refresh
```

### Advanced Terraform Commands
```bash
# Validate configuration
terraform validate

# Format configuration files
terraform fmt

# Show current state
terraform show

# Output specific values
terraform output
terraform output azure_vpn_gateway_ips
terraform output -json > outputs.json
```

## Azure CLI Commands

### Resource Management
```bash
# List resource groups
az group list --output table

# Show specific resource group
az group show --name rg-vpn-demo

# List resources in group
az resource list --resource-group rg-vpn-demo --output table
```

### VPN Monitoring Commands
```bash
# Check VPN connection status
az network vpn-connection show \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-connection-gcp-t1-dev \
  --query connectionStatus

az network vpn-connection show \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-connection-gcp-t2-dev \
  --query connectionStatus

# List all VPN connections
az network vpn-connection list \
  --resource-group rg-vpn-demo \
  --output table

# Check VPN gateway status
az network vnet-gateway show \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-gateway-dev \
  --query provisioningState

# Get BGP peer status
az network vnet-gateway list-bgp-peer-status \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-gateway-dev

# List VPN gateway connections
az network vnet-gateway list-connections \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-gateway-dev
```

### Network Diagnostics
```bash
# Test network connectivity
az network watcher test-connectivity \
  --source-resource-group rg-vpn-demo \
  --source-resource test-vm-azure \
  --dest-address 10.2.1.4 \
  --dest-port 22

# Check route table
az network route-table route list \
  --resource-group rg-vpn-demo \
  --route-table-name vpn-demo-rt-dev \
  --output table
```

## Google Cloud Commands

### Project and Resource Management
```bash
# Show current project
gcloud config get-value project

# List compute resources
gcloud compute instances list
gcloud compute networks list
gcloud compute vpn-gateways list

# Show project information
gcloud compute project-info describe --project=terraform-vpn-464313
```

### VPN Monitoring Commands
```bash
# Check HA VPN gateway status
gcloud compute vpn-gateways describe vpn-demo-ha-vpn-gateway-dev \
  --region us-central1

# Check VPN tunnel status
gcloud compute vpn-tunnels describe vpn-demo-vpn-tunnel-azure-t1-dev \
  --region us-central1 \
  --format="value(status)"

gcloud compute vpn-tunnels describe vpn-demo-vpn-tunnel-azure-t2-dev \
  --region us-central1 \
  --format="value(status)"

# List all VPN tunnels
gcloud compute vpn-tunnels list --filter="region:us-central1"

# Check Cloud Router status
gcloud compute routers get-status vpn-demo-router-dev \
  --region us-central1

# Check BGP sessions
gcloud compute routers get-status vpn-demo-router-dev \
  --region us-central1 \
  --format="table(result.bgpPeerStatus[].name,result.bgpPeerStatus[].state)"
```

### Network Diagnostics
```bash
# Check firewall rules
gcloud compute firewall-rules list --filter="network:vpn-demo-vpc-dev"

# Describe specific firewall rule
gcloud compute firewall-rules describe vpn-demo-allow-azure-dev

# Check routes
gcloud compute routes list --filter="network:vpn-demo-vpc-dev"

# Test connectivity (if test VMs exist)
gcloud compute ssh test-vm-gcp --zone us-central1-a --command="ping -c 4 10.1.1.4"
```

## Connectivity Testing

### Basic Network Tests
```bash
# Test from Azure VM to GCP (if VMs deployed)
# First SSH to Azure VM
ssh azureuser@<azure-vm-public-ip>

# Then test connectivity to GCP
ping -c 4 10.2.1.4
traceroute 10.2.1.4
telnet 10.2.1.4 22

# Test from GCP VM to Azure
gcloud compute ssh test-vm-gcp --zone us-central1-a
ping -c 4 10.1.1.4
traceroute 10.1.1.4
```

### Advanced Connectivity Tests
```bash
# Test specific ports
nc -zv 10.2.1.4 22    # Test SSH port
nc -zv 10.2.1.4 80    # Test HTTP port
nc -zv 10.2.1.4 443   # Test HTTPS port

# Bandwidth testing (install iperf3 first)
# On target machine
iperf3 -s

# On source machine
iperf3 -c 10.2.1.4 -t 30
```

## Monitoring and Logging

### Azure Monitoring
```bash
# Get VPN gateway metrics
az monitor metrics list \
  --resource /subscriptions/7d939770-deef-433d-9283-5bd5eb79aeaf/resourceGroups/rg-vpn-demo/providers/Microsoft.Network/virtualNetworkGateways/vpn-demo-vpn-gateway-dev \
  --metric "TunnelAverageBandwidth"

# Check activity logs
az monitor activity-log list \
  --resource-group rg-vpn-demo \
  --max-events 50
```

### GCP Monitoring
```bash
# Check VPN metrics
gcloud logging read "resource.type=vpn_gateway" --limit=50

# Monitor tunnel logs
gcloud logging read "resource.type=vpn_tunnel AND resource.labels.tunnel_name=vpn-demo-vpn-tunnel-azure-t1-dev" --limit=20
```

## File Operations

### Project File Management
```bash
# Create project directory
mkdir azure-gcp-vpn-terraform
cd azure-gcp-vpn-terraform

# Create module directories
mkdir -p modules/azure-networking
mkdir -p modules/gcp-networking

# Copy configuration files
cp terraform.tfvars.example terraform.tfvars

# Edit configuration
nano terraform.tfvars
nano modules/azure-networking/main.tf

# Check file permissions
ls -la terraform.tfvars
chmod 600 terraform.tfvars  # Secure sensitive files
```

### Backup and Version Control
```bash
# Initialize git repository
git init
git add .
git commit -m "Initial VPN configuration"

# Create backup
tar -czf vpn-backup-$(date +%Y%m%d).tar.gz .

# Backup Terraform state
cp terraform.tfstate terraform.tfstate.backup
```

## Troubleshooting Commands

### Debug Terraform Issues
```bash
# Enable detailed logging
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform.log
terraform apply

# Disable logging
unset TF_LOG
unset TF_LOG_PATH

# Check for syntax errors
terraform validate
terraform fmt -check
```

### Network Troubleshooting
```bash
# Check DNS resolution
nslookup vpn-demo-vpn-gateway-dev.eastus.cloudapp.azure.com
dig vpn-demo-vpn-gateway-dev.eastus.cloudapp.azure.com

# Check routing
ip route show
route -n

# Check network interfaces
ip addr show
ifconfig
```

### System Resource Monitoring
```bash
# Check system resources
top
htop
free -h
df -h

# Monitor network traffic
netstat -tuln
ss -tuln
iftop  # If installed
```