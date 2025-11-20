# 🧩 Technologies Used  
## Azure ↔ GCP High Availability (HA) VPN Infrastructure

This document explains **what each technology is**, **why it is used**, and **how it works within this project**. It is written for architects, engineers, and hiring managers evaluating multi-cloud networking and security capabilities.

---

# 1. Microsoft Azure Technologies

## 1.1 Azure Virtual Network (VNet)
**What it is:**  
Azure’s foundational private network boundary that hosts subnets, virtual machines, gateways, and routing components.

**Why it’s used:**  
To provide an isolated address space for workloads that will communicate with Google Cloud through HA VPN.

**How it works here:**  
The VNet contains a **GatewaySubnet** that hosts the Azure VPN Gateway in active/active mode.

---

## 1.2 Azure VPN Gateway (Active/Active)
**What it is:**  
A managed IPSec VPN service that supports site-to-site connections with high availability.

**Why it’s used:**  
Azure’s VPN Gateway in **active/active** mode provides two gateway instances, enabling multiple redundant tunnels to GCP.

**How it works here:**  
- Deploys two VPN Gateway instances  
- Each instance has its own public IP  
- Each instance establishes IPSec tunnels to GCP HA VPN interfaces  
- Supports dynamic routing (BGP)

---

## 1.3 Public IP Addresses
**What they are:**  
Static public IP resources assigned to the Azure VPN Gateway.

**Why used:**  
Required for GCP to establish inbound IPSec tunnels to Azure’s gateway instances.

**How used here:**  
Azure Gateway generates **two public IPs**, one per active instance—supporting full HA mesh tunnel creation.

---

## 1.4 Local Network Gateway
**What it is:**  
Azure’s representation of an external VPN endpoint.

**Why used:**  
Required to define GCP’s HA VPN gateway IPs and advertised prefixes.

**How it works here:**  
Each GCP interface is mapped as a local network gateway to connect Azure tunnels to specific GCP IP endpoints.

---

# 2. Google Cloud Technologies

## 2.1 Google Cloud VPC
**What it is:**  
GCP’s global private network for hosting workloads and subnets.

**Why used:**  
Acts as the GCP network side of the Azure ↔ GCP connection.

**How it works here:**  
The VPC hosts subnets reachable through the HA VPN and advertised through BGP.

---

## 2.2 Google Cloud HA VPN Gateway
**What it is:**  
GCP’s high-availability IPSec VPN service offering **two gateway interfaces**, each capable of creating two tunnels.

**Why used:**  
To build a true high-availability connection with redundancy across:
- Interfaces  
- Tunnels  
- Physical gateways  

**How it works here:**  
- Interface 0 → two tunnels to Azure gateway IPs  
- Interface 1 → two tunnels to Azure gateway IPs  
Creates **four tunnels total** in full mesh.

---

## 2.3 Google Cloud Router (BGP)
**What it is:**  
A managed BGP router that dynamically exchanges routes with external peers.

**Why used:**  
Enables dynamic routing and automatic failover without manual route management.

**How it works here:**  
- Cloud Router peers with Azure VPN Gateway instances  
- Advertises GCP subnets to Azure  
- Accepts Azure prefixes for return traffic  
- Reacts to tunnel failures instantly

---

# 3. Networking & Security Protocols

## 3.1 IPSec  
**What it is:**  
Industry-standard encryption protocol for site-to-site VPN tunnels.

**Why used:**  
Provides end-to-end encryption for all traffic between Azure and GCP.

**How used:**  
All four tunnels between Azure and GCP are IPSec/IKEv2 encrypted.

---

## 3.2 IKEv2  
**What it is:**  
A secure key exchange protocol used by IPSec VPN tunnels.

**Why used:**  
Provides faster negotiation, improved reconnection, and better HA behavior compared to IKEv1.

**How used:**  
Used for all tunnel negotiations across the Azure ↔ GCP mesh.

---

## 3.3 BGP (Border Gateway Protocol)
**What it is:**  
A dynamic routing protocol used across large networks.

**Why used:**  
- Automatically advertises prefixes  
- Handles failover without manual route changes  
- Ensures routes stay consistent across multiple tunnels

**How used:**  
Each Azure gateway instance forms BGP sessions with each GCP Cloud Router interface.

---

# 4. Terraform (Infrastructure-as-Code)

## 4.1 Terraform CLI  
**What it is:**  
Infrastructure-as-code tool for declaratively building multi-cloud infrastructure.

**Why used:**  
Ensures reproducible deployments in Azure and GCP with version-controlled configuration.

**How used:**  
- Creates Azure VPN Gateway, public IPs, VNets  
- Creates GCP VPC, HA VPN, Cloud Router  
- Configures tunnels, routing, and BGP  
- Supports teardown (`terraform destroy`) to minimize lab costs

---

## 4.2 Terraform AzureRM Provider
**What it is:**  
Terraform plugin for managing Azure resources.

**Why used:**  
Required to deploy Azure VNet, VPN Gateway, GatewaySubnet, etc.

---

## 4.3 Terraform Google Provider
**What it is:**  
Terraform plugin for managing GCP resources.

**Why used:**  
Required to deploy HA VPN, Cloud Router, and GCP VPC.

---

# 5. CLI Tools

## 5.1 Azure CLI  
Used for:  
- Verifying gateway configuration  
- Inspecting BGP sessions  
- Checking VPN connection health  

---

## 5.2 Google Cloud CLI (`gcloud`)  
Used for:  
- Troubleshooting HA VPN tunnels  
- Viewing BGP route advertisements  
- Fetching gateway and interface details  

---

# 6. Observability Technologies

## 6.1 Azure Log Analytics  
**Why used:**  
To monitor gateway logs such as:
- Tunnel state changes  
- IKE negotiation events  
- BGP session events  

---

## 6.2 Google Cloud Logging  
**Why used:**  
Captures HA VPN logs for:
- Tunnel up/down events  
- BGP updates  
- Tunnel negotiations and failures  

---

# 7. Optional Tools (for validation)

## 7.1 `ping` / `tracepath` / `traceroute`
Used to verify:
- Cross-cloud reachability  
- Failover paths  
- Latency  
- Route change behavior  

---

## 7.2 Netcat or HTTP Test Server
Used to validate application-level connectivity across the HA VPN.

---

# ✔️ Summary

This project uses a combination of **Azure native networking**, **GCP HA VPN**, **IPSec tunnels**, **BGP dynamic routing**, and **Terraform automation** to build a realistic, resilient multi-cloud network. These technologies demonstrate how a Cloud Security Architect designs secure, fault-tolerant, cloud-agnostic connectivity between major cloud platforms.

