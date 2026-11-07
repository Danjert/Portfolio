# Local Enterprise SOC Simulation Lab

## Overview
This directory houses the professional documentation, architecture specifications, configuration guides, and adversary simulations for a local enterprise-grade Security Operations Center (SOC) laboratory. Built within Oracle VirtualBox using an **OPNsense** core perimeter firewall and an **Ubuntu Enterprise Splunk SIEM** server, this environment provides a controlled, zero-trust landscape designed to practice network telemetry collection, log parsing, threat detection, and incident response.

---

## Lab Architecture & Network Topology

The laboratory enforces strict Layer 3 isolation across four dedicated virtual subnets using static IP assignments to ensure telemetry consistency:

* **WAN Boundary(`DHCP`):** Upstream internet connection (NAT/Bridged).
* **DMZ Network (`10.0.50.0/24`):** High-risk service zone hosting target endpoints (Windows 11 and Metasploitable). Outbound traffic is heavily restricted.
* **Core Management / LAN (`10.0.100.0/24`):** Secure management plane housing the central Splunk Enterprise server and firewall WebGUI.
* **Attacker Network (`10.0.200.0/24`):** Isolated attacker zone hosting Kali Linux, permitted to route into the DMZ while strictly blocked from the core management plane.

---

## Directory Structure & File Descriptions

```text
Local_Enterprise_SOC_Lab/
├── README.md                             # Repository overview & file index
├── Module1_Firewall_Deployment.md        # Perimeter provisioning & ACLs
├── Module2_Splunk_SIEM_Deployment.md     # (Under Construction) SIEM setup & syslog
├── Module3_Endpoint_Telemetry_Harden.md  # (Under Construction) Windows audit & Sysmon
└── Module4_Threat_Simulations/           # (Under Construction) Adversary simulations
    ├── 01_Reconnaissance_Port_Scanning.md
    ├── 02_Credential_Access_BruteForce.md
    └── 03_Web_Application_Exploitation.md
```

---

### File Breakdown

* **[Module 1 - Firewall Deployment](./Module1_Firewall_Deployment.md)**
  * **Contents:** Details the complete deployment of the OPNsense firewall virtual machine. It outlines virtual network interface card (vNIC) mapping, static IP addressing schemas across all four zones, and the comprehensive stateful firewall rule matrix (ACLs) enforcing network segmentation. It also documents initial syslog export validation.

* **`Module2_Splunk_SIEM_Deployment.md`**
  * **Contents:** (Under Construction)

* **`Module3_Endpoint_Telemetry_Harden.md`**
  * **Contents:** (Under Construction)

* **`Module4_Threat_Simulations/`**
  * **Contents:** (Under Construction)
