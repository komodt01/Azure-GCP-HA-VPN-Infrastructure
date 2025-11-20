# 🌐 Azure ↔ GCP High Availability (HA) VPN Infrastructure  
Secure Multi-Cloud Connectivity with IPSec, IKEv2, BGP, and Terraform

---

## 📌 Project Overview

This project implements a **high-availability site-to-site VPN** between **Microsoft Azure** and **Google Cloud Platform (GCP)** using:

- Azure VPN Gateway (Active/Active)
- GCP HA VPN (two interfaces, four tunnels)
- IPSec / IKEv2 encryption
- Dynamic routing with BGP
- Terraform for repeatable deployment
- Azure CLI + gcloud CLI for verification

The design replicates **enterprise-grade multi-cloud connectivity**, demonstrating Cloud Security Architecture skills in:

- Network resiliency  
- Secure cross-cloud routing  
- Encryption standards  
- Zero Trust networking principles  

This is a **lab-friendly** implementation, but it reflects real-world architecture patterns used in production environments.

---

## 📑 Documentation Included

| File | Description |
|------|-------------|
| `business-case.md` | Why this architecture matters (business drivers, risks, value) |
| `requirements.md` | Functional, technical, security, and operational requirements |
| `technologies.md` | What each Azure/GCP technology is and why it's used |
| `compliance_mapping.md` | NIST 800-53, ISO 27001, CIS, and Zero Trust alignment |
| `linux commands.md` | Full Azure, GCP, Terraform, and diagnostics command reference |

---

## 🧭 Architecture Summary

This project establishes a **4-tunnel redundant VPN mesh**:

```
Azure VPN Gateway (Active/Active)
    ↕ Tunnel 1,2
GCP HA VPN Interface 0
    ↕ Tunnel 3,4
GCP HA VPN Interface 1
```

Each Azure gateway instance peers with each GCP HA interface.  
Each tunnel carries a **BGP session**, enabling:

- Automated route learning  
- Failover without manual intervention  
- Dynamic path selection  

If any gateway instance or tunnel fails, **traffic automatically reroutes**.

---

## 🔐 Security Features

- AES-256 IPSec encryption for all cross-cloud traffic  
- IKEv2 for secure negotiation  
- BGP authenticated sessions  
- Least-privilege routing (only required CIDRs advertised)  
- Non-overlapping address spaces  
- No inbound public exposure except VPN endpoints  
- Terraform-managed configurations eliminate drift  
- Logging enabled in both clouds (Azure Monitor + Cloud Logging)  

---

## 📦 Technologies Used

- **Azure VPN Gateway** (Active/Active)
- **GCP HA VPN Gateway**
- **Azure Virtual Network (VNet)**
- **Google VPC**
- **IPSec / IKEv2**
- **BGP (Cloud Router + Azure Gateway)**
- **Terraform**
- **Azure CLI**
- **GCloud CLI**
- **Log Analytics / Cloud Logging**

See `technologies.md` for detailed explanations.

---

## 🏗️ Deployment Instructions

### 1. Authenticate to Azure and GCP

```bash
az login
az account set --subscription "<subscription-id>"

gcloud auth login
gcloud config set project <gcp-project-id>
gcloud auth application-default login
```

---

### 2. Enable required GCP APIs

```bash
gcloud services enable compute.googleapis.com iam.googleapis.com cloudresourcemanager.googleapis.com
```

---

### 3. Initialize Terraform

```bash
terraform init
terraform plan
terraform apply
```

To auto-approve:

```bash
terraform apply -auto-approve
```

---

## 📡 Verification Steps

### Check Azure VPN tunnel status

```bash
az network vnet-gateway list-bgp-peer-status \
  --resource-group <rg-name> \
  --name <gateway-name>
```

### Check GCP VPN tunnel status

```bash
gcloud compute vpn-tunnels list --filter="region:<region>"
```

### Validate BGP sessions

```bash
gcloud compute routers get-status <router-name> --region <region>
```

### End-to-end VM connectivity (optional)

From Azure VM:
```bash
ping -c 4 <gcp-vm-private-ip>
```

From GCP VM:
```bash
ping -c 4 <azure-vm-private-ip>
```

---

## 📊 Monitoring

### Azure

```bash
az monitor metrics list \
  --resource <gateway-resource-id> \
  --metric "TunnelAverageBandwidth"
```

### GCP

```bash
gcloud logging read "resource.type=vpn_gateway" --limit=20
```

---

## 🧹 Teardown (Avoiding Costs)

```bash
terraform destroy
```

---

## 📁 Recommended Repository Structure

```
azure-gcp-ha-vpn/
├── README.md
├── business-case.md
├── requirements.md
├── technologies.md
├── compliance_mapping.md
├── linux commands.md
├── terraform/
│   ├── azure/
│   ├── gcp/
│   └── modules/
└── diagrams/
    └── ha-vpn-architecture.png
```

---

## 🎯 Project Value (for Recruiters & Hiring Managers)

This project demonstrates:

- Secure multi-cloud design  
- Cross-cloud routing & encryption  
- High availability network engineering  
- Terraform IaC best practices  
- Cloud security architecture skills  
- Hands-on Azure + GCP operational competency  
- Zero Trust-aligned network segmentation  
- BGP troubleshooting and resilience testing  

---

## ✔️ Next Steps (Optional Enhancements)

- Add Cloud NAT and internet egress controls  
- Integrate Azure Firewall or GCP Cloud Armor  
- Deploy test workloads for performance measurement  
- Add a hub-and-spoke design to each cloud  
- Implement a third cloud (AWS) for full multi-cloud mesh  



