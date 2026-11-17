# TakeOver - Cloud Subdomain Takeover & Asset Visibility Assessment

## Overview
This directory contains a multi-perspective security assessment of the **TakeOver** Room (TryHackMe lab). The scenario evaluates external perimeter exposure, virtual host enumeration, certificate analysis, and the risk of forgotten DNS records or redirects that point to unmanaged cloud storage (AWS S3).

> Performed only in an authorized lab environment. The flag value and the hidden hostname are masked in these documents.

## Repository Structure

| Document | Audience | Contents |
| :--- | :--- | :--- |
| [Executive Report](./Executive_Report.md) | Leadership, GRC | Business impact briefing, findings with a rating method and evidence references, Cyber Kill Chain and MITRE ATT&CK alignment, NIST CSF 2.0 and SP 800-53 mappings, recommendations and verification steps |
| [Technical Walkthrough](./TakeOver.md) | SOC and penetration testing teams | Step-by-step methodology, commands, virtual host discovery, certificate analysis and redirect verification |

## Key Takeaways & Skills Demonstrated
- **Attack surface management:** Identification of hidden hostnames and redirects to unmanaged cloud endpoints.
- **Enumeration:** Virtual host discovery, SSL/TLS certificate inspection.
- **Framework alignment:** Mapping to the Cyber Kill Chain, MITRE ATT&CK and NIST, distinguishing between lab findings and potential real-world impact.
- **Reporting:** Separate executive and technical deliverables with a stated rating method and evidence references.
