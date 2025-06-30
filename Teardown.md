# Infrastructure Teardown Guide

## Overview

This guide provides comprehensive instructions for safely removing the Azure-GCP HA VPN infrastructure. Follow these steps to avoid unexpected costs and ensure complete cleanup.

## Pre-Teardown Checklist

### 1. **Backup Important Data**
```bash
# Backup Terraform state
cp terraform.tfstate terraform.tfstate.backup.$(date +%Y%m%d)

# Export current configuration
terraform show > infrastructure-snapshot.txt

# Save outputs for reference
terraform output > final-outputs.txt
```

### 2. **Document Current State**
```bash
# List all managed resources
terraform state list > managed-resources.txt

# Check Azure resources
az resource list --resource-group rg-vpn-demo --output table > azure-resources.txt

# Check GCP resources  
gcloud compute instances list --project=terraform-vpn-464313 > gcp-resources.txt
```

### 3. **Verify No Critical Dependencies**
- Confirm no production workloads depend on the VPN connection
- Check for any automated processes using the connectivity
- Verify no other infrastructure references these resources

## Teardown Methods

## Method 1: Terraform Destroy (Recommended)

### Complete Infrastructure Removal
```bash
# Navigate to project directory
cd azure-gcp-vpn-terraform

# Plan the destruction (review what will be deleted)
terraform plan -destroy

# Execute destruction with confirmation
terraform destroy

# Type 'yes' when prompted to confirm
```

### Selective Resource Removal
```bash
# Remove specific modules first (if needed)
terraform destroy -target=module.gcp_networking
terraform destroy -target=module.azure_networking

# Remove remaining resources
terraform destroy
```

## Method 2: Manual Azure Cleanup

### If Terraform Destroy Fails for Azure Resources
```bash
# Delete VPN connections first
az network vpn-connection delete \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-connection-gcp-t1-dev

az network vpn-connection delete \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-connection-gcp-t2-dev

# Delete Local Network Gateways
az network local-gateway delete \
  --resource-group rg-vpn-demo \
  --name vpn-demo-lng-gcp-t1-dev

az network local-gateway delete \
  --resource-group rg-vpn-demo \
  --name vpn-demo-lng-gcp-t2-dev

# Delete VPN Gateway (takes 10-15 minutes)
az network vnet-gateway delete \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-gateway-dev

# Delete Public IPs
az network public-ip delete \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-pip-1-dev

az network public-ip delete \
  --resource-group rg-vpn-demo \
  --name vpn-demo-vpn-pip-2-dev