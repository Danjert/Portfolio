# Cybersecurity Portfolio

Welcome to my cybersecurity portfolio repository 

The purpose of this repository is to showcase practical cybersecurity projects and technical assessments. Here, you will find documentation highlighting:

- **Technical Tool Proficiency:** Practical applications of tools and core command-line utilities.
- **Methodical Problem-Solving:** Detailed, step-by-step documentation.
- **Communication & Reporting:** Clean, professional Markdown write-ups designed to translate technical findings into actionable insights.

---

## Featured Walkthroughs & Projects

### Local Enterprise SOC Environment
* **[Module 1 - Firewall Deployment](./Local_Enterprise_SOC_Lab/Module1_Firewall_Deployment.md)**
  * **Overview:** This documentation details the design, deployment, and validation of an enterprise-grade OPNsense perimeter firewall, featuring multi-zone network segmentation, static IP architecture, explicit stateful filtering matrices, and Splunk SIEM syslog telemetry verification.

* **[Module 2 - Splunk SIEM Deployment](./Local_Enterprise_SOC_Lab/Module_2_Splunk_Deployment.md)**
  * **Overview:** This documentation covers the provisioning, configuration, and hardening of an Ubuntu Server running Splunk Enterprise within the SOC lab. It highlights least-privilege service account management, OS performance tuning (swap, sysctl, file descriptors, UFW), dedicated index creation (`opnsense`), UDP syslog ingestion (`5140`), and the deployment of the Splunk Add-on for OPNsense (`TA-opnsense`) for Common Information Model (CIM) field normalization and end-to-end pipeline verification.

* **[Module 3 - Windows Endpoint & Telemetry Onboarding](./Local_Enterprise_SOC_Lab/Module_3_WindowsEndpoint_Deployment.md)**
  * **Overview:** This documentation covers the provisioning, configuration, and hardening of a Windows 11 Enterprise endpoint within the SOC lab's DMZ segment. It highlights advanced local Windows audit policy hardening (Event ID 4688 with command-line arguments and PowerShell script block logging 4104), Microsoft Sysmon deployment utilizing the hardened SwiftOnSecurity configuration, Splunk Universal Forwarder configuration for secure TCP `9997` telemetry routing, and end-to-end event pipeline verification.

* **[Module 4 - Suricata IDS](./Local_Enterprise_SOC_Lab/Module_4_IDS_Deployment.md)**
  * **Overview:** This documentation details the deployment, configuration, and integration of the Suricata Intrusion Detection System directly onto the OPNsense perimeter firewall (`10.0.100.1`). It highlights passive PCAP live mode configuration, promiscuous mode interface mapping across segmented zones, Hyperscan pattern-matching optimization, Emerging Threats ruleset provisioning, and EVE JSON syslog streaming routed to Splunk Enterprise (`10.0.100.10`) for automated field extraction and network threat visibility.

### TryHackMe
* **[Corridor - IDOR Vulnerability Assessment](./TryHackMe/Corridor)**
  * **Overview:** This directory contains the professional security documentation for the Corridor Room assessment, featuring two files: the technical walkthrough, which provides a step-by-step breakdown of the endpoint enumeration, hash reversal, and boundary traversal methodology, and the executive report, which delivers a formal risk assessment mapped to enterprise frameworks including NIST, MITRE ATT&CK, and OWASP Top 10:2025.

* **[TakeOver - Subdomain Takeover Assessment](./TryHackMe/TakeOver/)**
  * **Overview:** This directory contains the professional security documentation for the Takeover Room assessment, featuring two files: the technical walkthrough, which provides a step-by-step breakdown of subdomain enumeration via Gobuster, SSL/TLS certificate analysis, and orphaned DNS CNAME record verification, and the executive report, which delivers a formal risk assessment mapped to enterprise frameworks including NIST, MITRE ATT&CK, and the Cyber Kill Chain.

---

##  Connect

[LinkedIn](https://www.linkedin.com/in/daniel-alejandro-jerez-torres/)
[GitHub](https://github.com/Danjert)
