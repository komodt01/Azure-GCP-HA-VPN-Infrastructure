# 🛡️ Compliance Mapping  
## Azure ↔ GCP High Availability (HA) VPN Infrastructure

This document maps the project’s architecture and controls to major security and compliance frameworks commonly referenced in cloud security architecture: **NIST 800-53**, **ISO 27001:2022**, **CIS Benchmarks**, and **Zero Trust** principles.

---

# 1. NIST 800-53 Compliance Mapping

| Control | Title | How This Project Meets the Control |
|---------|--------|------------------------------------|
| **SC-7** | Boundary Protection | VPN gateways create secure network boundaries between Azure and GCP; IPSec tunnels enforce encrypted cross-cloud communication. |
| **SC-7(12)** | Access Points | Traffic flows are restricted to VPN endpoints and required CIDR ranges only (least privilege routing). |
| **SC-13** | Cryptographic Protection | Enables IPSec/IKEv2 encryption for all inter-cloud traffic. |
| **SC-20** | Secure Name/Address Resolution | Azure and GCP resources use cloud-native DNS with private address spaces. |
| **SC-21** | Secure Name/Address Resolution (Authenticated) | BGP uses authenticated routing and validated peers. |
| **AC-4** | Information Flow Enforcement | Route filtering and isolated networks enforce controlled information flow between Azure and GCP. |
| **CM-2** | Baseline Configuration | Terraform IaC provides consistent, documented configuration. |
| **CM-6** | Configuration Settings | Terraform modules enforce secure network configuration baselines for both clouds. |
| **CA-3** | System Interconnections | The Azure ↔ GCP VPN interconnection is documented and controlled with encryption and BGP. |
| **CP-10** | System Recovery & Continuity | HA tunnels and redundant gateways provide continuity during failures. |
| **SI-4** | System Monitoring | Azure Log Analytics and GCP Cloud Logging provide tunnel and route monitoring. |

---

# 2. ISO 27001:2022 Compliance Mapping

| Clause | Title | Alignment |
|--------|--------|-----------|
| **A.5.19** | Secure network architecture | HA VPN design ensures secure architecture between clouds with redundancy. |
| **A.8.20** | Network security | IPSec encryption, non-overlapping CIDRs, and BGP authentication meet network security best practices. |
| **A.8.16** | Monitoring activities | Azure Monitor + GCP Logging track tunnel health and routing. |
| **A.12.1** | Change management | Terraform provides versioned, controlled infrastructure changes. |
| **A.12.4** | Logging and monitoring | Logs from both gateways provide audit data. |
| **A.14.1** | Security requirements of information systems | Defined requirements for encryption, routing, and HA connectivity. |
| **A.14.2** | Secure development | Infrastructure-as-Code development practices enforce consistent, secure deployment. |
| **A.5.30** | ICT readiness for continuity | HA tunnels, failover scenarios, and BGP dynamic routing provide continuity across clouds. |

---

# 3. CIS Benchmarks Mapping  
### CIS Microsoft Azure Foundations Benchmark  
| CIS Control | Description | Implementation |
|-------------|-------------|----------------|
| **1.2** | Ensure secure network design | Uses private VNets, GatewaySubnet, least-privilege routing. |
| **5.1** | Network security groups | Optional NSG validation for test VMs. |
| **4.1** | Logging retention | Logs sent to Log Analytics. |
| **3.2** | Ensure secure access to gateways | Azure VPN Gateway is deployed with secure tunneling protocols. |

### CIS Google Cloud Foundations Benchmark  
| CIS Control | Description | Implementation |
|-------------|-------------|----------------|
| **3.7** | VPN tunnels secured | GCP HA VPN uses IPSec/IKEv2. |
| **2.8** | Ensure VPC flow logs enabled | Logging enabled for HA VPN and VPC. |
| **4.1** | Restrict network ingress | Only IPSec/IKE traffic allowed based on gateway endpoints. |

---

# 4. Zero Trust Architecture Mapping

| Zero Trust Principle | Alignment |
|----------------------|-----------|
| **Encrypt Everything** | IPSec tunnels encrypt all inter-cloud traffic. |
| **Assume Breach** | No trust is placed on public internet paths; only authenticated gateways allowed. |
| **Least Privilege Access** | Only required prefixes exchanged between clouds; no broad routing allowed. |
| **Continuous Monitoring** | Azure + GCP logs show tunnel health, BGP state, and negotiation events. |
| **Strong Identity for Network Paths** | Tunnels authenticated via pre-shared keys or certificates; BGP peers validated. |
| **Micro-Segmentation** | VNets/VPCs isolate networks; only approved subnets are reachable. |

---

# 5. Audit Evidence Produced by This Project

This project generates the following artifacts suitable for compliance audits:

- Terraform code (version-controlled configuration baseline)  
- Architecture diagrams (system interconnection documentation)  
- Logs from both clouds showing:  
  - Tunnel up/down events  
  - IKE negotiations  
  - BGP session states  
  - Route advertisements  
- Output of cross-cloud connectivity tests (evidence of flow control)  
- Documentation (business case, requirements, technologies, compliance mapping)

These artifacts align with audit requirements under SOC 2, ISO 27001, PCI DSS, NIST CSF, and internal governance frameworks.

---

# ✔️ End of Compliance Mapping Document
