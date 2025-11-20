# 🌐 Azure ↔ GCP High Availability (HA) VPN Infrastructure  
Secure Multi-Cloud Connectivity with IPSec, IKEv2, BGP, and Native Cloud Tools

---

## 📌 Project Overview

This project implements a **high-availability site-to-site VPN** between **Microsoft Azure** and **Google Cloud Platform (GCP)** using only:

- Azure VPN Gateway (Active/Active)
- GCP HA VPN (two interfaces, four tunnels)
- IPSec / IKEv2 encryption
- Dynamic routing with BGP
- Azure CLI and gcloud CLI for configuration and validation

The design models **enterprise-grade multi-cloud connectivity** and demonstrates Cloud Security Architecture skills in:

- Network resiliency and redundancy  
- Secure cross-cloud routing  
- Encryption standards (IPSec / IKEv2)  
- Zero Trust network segmentation  

This is a **home lab**, but the architecture patterns mirror the way multi-cloud backbones are built in real environments.

---

## 📑 Documentation Included

| File                    | Description |
|-------------------------|-------------|
| `business-case.md`      | Why the Azure ↔ GCP HA VPN exists, business drivers, value, and risks |
| `requirements.md`       | Business, functional, technical, networking, and security requirements |
| `technologies.md`       | Explanation of Azure, GCP, IPSec, BGP, and monitoring technologies used |
| `compliance_mapping.md` | Mapping to NIST 800-53, ISO 27001, CIS Benchmarks, and Zero Trust |
| `linux_commands.md`     | Detailed Azure CLI, gcloud CLI, and troubleshooting command reference |

---

## 🧭 Architecture Summary

The project builds a **highly available cross-cloud VPN mesh**:

- **Azure side**
  - Azure VNet with a `GatewaySubnet`
  - Azure VPN Gateway in **Active/Active** mode
  - Two public IPs (one per gateway instance)

- **GCP side**
  - GCP VPC network
  - GCP HA VPN Gateway with **two interfaces**
  - Cloud Router for BGP

- **Tunnels and Routing**
  - Four IPSec tunnels in total:
    - Azure Gateway Instance 1 ↔ GCP Interface 0  
    - Azure Gateway Instance 1 ↔ GCP Interface 1  
    - Azure Gateway Instance 2 ↔ GCP Interface 0  
    - Azure Gateway Instance 2 ↔ GCP Interface 1  
  - BGP sessions over each tunnel  
  - Non-overlapping CIDR ranges in Azure and GCP  
  - Automatic failover via dynamic routing

If any tunnel or gateway instance fails, BGP reconverges and traffic continues over the remaining tunnels.

---

## 🔐 Security Features

- **End-to-end encryption** using IPSec / IKEv2
- **Dynamic routing** with BGP (no static route sprawl)
- **Least-privilege routing** (only required prefixes advertised)
- **No overlapping address spaces** between Azure and GCP
- **No direct public exposure** of workloads—only VPN endpoints have public IPs
- **Logging and monitoring** of tunnel health and routing:
  - Azure: Log Analytics / Activity Logs
  - GCP: Cloud Logging

---

## 🧩 Technologies Used

Key technologies (see `technologies.md` for full details):

- **Azure Virtual Network (VNet)**
- **Azure VPN Gateway (Active/Active)**
- **Azure Public IP Addresses**
- **Google Cloud VPC**
- **Google Cloud HA VPN Gateway**
- **Cloud Router (BGP)**
- **IPSec / IKEv2**
- **BGP Routing**
- **Azure CLI**
- **gcloud CLI**
- **Azure Monitor / Log Analytics**
- **GCP Cloud Logging**

---

## 🧱 Prerequisites

- Azure Subscription with permission to create:
  - Resource groups
  - VNets
  - VPN gateways
- GCP Project with permission to create:
  - VPC networks
  - HA VPN gateways
  - Cloud Routers
- Installed locally:
  - [Azure CLI](https://learn.microsoft.com/cli/azure/)
  - [Google Cloud SDK (gcloud)](https://cloud.google.com/sdk)
  - SSH client (for VM tests)
  - Optional: `traceroute`, `netcat`, `iperf3` for deeper diagnostics

---

## 🚀 High-Level Deployment Steps (Portal + CLI)

### 1. Authenticate to Azure and GCP

```bash
# Azure
az login
az account set --subscription "<subscription-id>"
az account show

# GCP
gcloud auth login
gcloud config set project <gcp-project-id>
gcloud auth application-default login
```

---

### 2. Azure Networking and VPN Gateway

1. Create a **Resource Group** (portal or CLI).  
2. Create a **VNet** with non-overlapping CIDR (e.g., `10.1.0.0/16`).  
3. Add a **GatewaySubnet** (required by Azure VPN Gateway).  
4. Create an **Azure VPN Gateway** in **Active/Active** mode with VPN type “Route-based.”  
5. Assign **two static public IPs** to the gateway (one per instance).

You can verify resources using:

```bash
az group list --output table
az resource list --resource-group <rg-name> --output table
az network vnet-gateway show \
  --resource-group <rg-name> \
  --name <gateway-name>
```

---

### 3. GCP Networking and HA VPN

1. Create a **VPC network** with non-overlapping CIDR (e.g., `10.2.0.0/16`).  
2. Create a **subnet** for test workloads.  
3. Create a **Cloud Router** (BGP ASN configured).  
4. Create a **HA VPN Gateway** with **two interfaces**.  
5. Configure tunnels to Azure gateway IPs:
   - Each interface on GCP connects to both Azure gateway public IPs.  
   - Shared secrets/PSKs configured on both sides.

You can verify resources using:

```bash
gcloud config get-value project
gcloud compute networks list
gcloud compute vpn-gateways list
gcloud compute routers list
```

---

### 4. Configure IPSec Tunnels and BGP

On **Azure**:

- Define connections to each GCP endpoint using:
  - GCP interface public IPs
  - Shared keys
  - Azure BGP settings (ASN, peer IPs)

On **GCP**:

- Define four VPN tunnels:
  - Each tunnel associates with Cloud Router via BGP
  - BGP peers configured with Azure VPN Gateway BGP IP/ASN

Check BGP status:

```bash
# Azure
az network vnet-gateway list-bgp-peer-status \
  --resource-group <rg-name> \
  --name <gateway-name>

# GCP
gcloud compute routers get-status <router-name> \
  --region <region>
```

---

### 5. (Optional) Deploy Test VMs

- Create one **test VM in Azure** (in the connected VNet).  
- Create one **test VM in GCP** (in the connected VPC).  

Test connectivity:

From **Azure VM**:

```bash
ping -c 4 <gcp-vm-private-ip>
traceroute <gcp-vm-private-ip>
```

From **GCP VM**:

```bash
ping -c 4 <azure-vm-private-ip>
traceroute <azure-vm-private-ip>
```

---

## 📡 Monitoring & Troubleshooting

Many of these commands are already captured in `linux_commands.md`. Below are some key examples.

### Azure VPN Monitoring

```bash
# Check VPN connection status
az network vnet-gateway list-connections \
  --resource-group <rg-name> \
  --name <gateway-name>

# Check BGP peer status
az network vnet-gateway list-bgp-peer-status \
  --resource-group <rg-name> \
  --name <gateway-name>
```

### GCP VPN Monitoring

```bash
# Check HA VPN gateway
gcloud compute vpn-gateways describe <gateway-name> \
  --region <region>

# Check VPN tunnels
gcloud compute vpn-tunnels list --filter="region:<region>"

# Check Cloud Router BGP status
gcloud compute routers get-status <router-name> \
  --region <region> \
  --format="table(result.bgpPeerStatus[].name,result.bgpPeerStatus[].state)"
```

---

## 📁 Recommended Repository Structure

```text
azure-gcp-ha-vpn/
├── README.md
├── business-case.md
├── requirements.md
├── technologies.md
├── compliance_mapping.md
├── linux_commands.md
└── diagrams/
    └── ha-vpn-architecture.png
```

---

## 🎯 Project Value (for Recruiters & Hiring Managers)

This project demonstrates:

- **Multi-cloud network design** (Azure + GCP)  
- **High availability** through redundant gateways and tunnels  
- **Strong security posture** with IPSec/IKEv2 and least-privilege routing  
- **Operational competence** with Azure CLI and gcloud CLI  
- **Understanding of BGP and dynamic routing**  
- **Alignment to NIST / ISO / Zero Trust** (see `compliance_mapping.md`)  

It highlights your ability to design and operate secure, resilient, multi-cloud connectivity—core to Cloud Security Architect and Cloud Architect roles.

---

## ✔️ Next Potential Enhancements

- Add **Azure Firewall** or **GCP firewall rules** for deeper segmentation  
- Integrate centralized **logging dashboards** (Azure Monitor Workbook, GCP Metrics Explorer)  
- Extend design into a hub-and-spoke topology on each side  
- Explore **latency and throughput** testing with `iperf3` across the tunnel  

---

## ✅ Status

This README reflects a **CLI/portal-based** Azure ↔ GCP HA VPN project **without Terraform**.  
Terraform can be added later in a separate branch or future version if desired.

