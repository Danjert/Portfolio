# TakeOver — Cloud Subdomain Takeover & Asset Visibility Assessment

## Overview
This directory contains a multi-perspective security assessment of the **TakeOver** challenge. The scenario evaluates external perimeter exposure, virtual host enumeration, certificate analysis, and the risk of orphaned cloud infrastructure (forgotten DNS records pointing to unmanaged AWS S3 buckets).

## Repository Structure
* [Executive Report & GRC Mapping (Executive-Report.md)](./Executive_Report.md) - Strategic business impact briefing, NIST CSF/SP 800-53 control mappings, Cyber Kill Chain and MITRE ATT&CK TTP alignment for leadership and GRC teams.
* [Technical Walkthrough & Verification (Walkthrough.md)](./TakeOver.md) - Step-by-step technical methodology, commands, terminal output, and flag verification steps for SOC and penetration testing teams.

## Key Takeaways & Skills Demonstrated
* **Attack Surface Management:** Identification of orphaned DNS CNAME records and unmanaged cloud endpoints.
* **Enumeration:** Vhost discovery, HTTP header analysis, and SSL/TLS inspection.
* **Framework Alignment:** Practical mapping across Lockheed Martin Cyber Kill Chain, MITRE ATT&CK, and NIST standards.
