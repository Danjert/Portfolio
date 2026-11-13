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
├── README.md                               # Repository overview & file index
├── Module_1_Firewall_Deployment.md         # Perimeter provisioning & ACLs
├── Module_2_Splunk_Deployment.md           # SIEM setup, indexing & OPNsense TA
├── Module_3_WindowsEndpoint_Deployment.md  # Windows logs & Sysmon
├── Module_4_IDS_Deployment.md              # Suricata IDS Deployment
├── Module_5_WazuhEDR_Deployment.md         # (Under Construction)
└── Module_6_Threat_Simulations/            # (Under Construction) Adversary simulations
    ├── 01_Reconnaissance_Port_Scanning.md
    ├── 02_Credential_Access_BruteForce.md
    └── 03_Web_Application_Exploitation.md
```

---

### File Breakdown

* **[Module 1 - Firewall Deployment](./Module1_Firewall_Deployment.md)**
  * **Contents:** Details the complete deployment of the OPNsense firewall virtual machine. It outlines virtual network interface card (vNIC) mapping, static IP addressing schemas across all four zones, and the comprehensive stateful firewall rule matrix (ACLs) enforcing network segmentation. It also documents initial syslog export validation.

* **[Module 2 - Splunk Deployment](./Module_2_Splunk_Deployment.md)**
  * **Contents:** Documents the provisioning and hardening of the central Ubuntu Server running Splunk Enterprise (`10.0.100.10`). Covers OS performance tuning (4GB swap, limits.conf, sysctl), unprivileged service account management, UFW firewall rules, dedicated `opnsense` index creation, UDP syslog listener (`5140`), and the installation/configuration of the Splunk Add-on for OPNsense (`TA-opnsense`) for CIM-compliant field normalization.

* **[Module 3 - Windows Endpoint Deployment](./Module_3_WindowsEndpoint_Deployment.md)**
  * **Contents:** Details the provisioning and onboarding of the Windows 11 Enterprise endpoint within the DMZ network segment (`10.0.50.100`). It covers advanced Windows audit policy hardening (Event IDs 4688 with command-line auditing and PowerShell script block logging 4104), Microsoft Sysmon deployment utilizing the hardened SwiftOnSecurity configuration, and the configuration and deployment of the Splunk Universal Forwarder to securely stream structured XML logs and Sysmon telemetry over TCP 9997 to the central Splunk indexer.

* **[Module 4 - Suricata IDS Deployment](./Module_4_IDS_Deployment.md)**
  * **Contents:** Details the deployment and configuration of the Suricata Intrusion Detection System directly onto the OPNsense perimeter firewall (`10.0.100.1`). Covers PCAP live mode configuration, promiscuous mode mapping across segmented interfaces, Hyperscan pattern-matching optimization, Emerging Threats ruleset provisioning, and EVE JSON syslog streaming routed to Splunk (`10.0.100.10`) for automated field extraction and network threat monitoring.

* **`Module_5_WazuhEDR_Deployment`**
  * **Contents:** (Under Construction)

* **`Module6_Threat_Simulations/`**
  * **Contents:** (Under Construction)
