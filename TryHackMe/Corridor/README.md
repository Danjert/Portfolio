# Corridor - Web Application Access Control & Cryptographic Failure Assessment

## Overview
This directory contains a multi-perspective security assessment of the **Corridor** web application environment (TryHackMe lab). The scenario evaluates the integrity of resource identifiers, server-side authorization, and the risks of using weak, reversible hashing to hide predictable identifiers.

> Performed only in an authorized lab environment. The flag value is not reproduced in these documents.

## Repository Structure

| Document | Audience | Contents |
| :--- | :--- | :--- |
| [Executive Report](./Executive_Report.md) | Leadership, GRC | Business impact, severity rating method, findings with evidence references, NIST CSF 2.0 and SP 800-53 mappings, MITRE ATT&CK alignment, OWASP Top 10:2025 integration, remediation and verification steps |
| [Technical Walkthrough](./Corridor.md) | SOC and penetration testing teams | Step-by-step methodology, commands, hash analysis and flag verification |

## Key Takeaways & Skills Demonstrated
- **Web application security:** Identification of missing server-side authorization on predictable direct object references (IDOR).
- **Enumeration & cryptographic analysis:** Endpoint discovery, hash identification, offline hash reversal.
- **Framework alignment:** Mapping findings to NIST, MITRE ATT&CK, and the OWASP Top 10:2025, distinguishing between lab findings and potential real-world impact.
- **Reporting:** Separate executive and technical deliverables with a stated rating method and evidence references.
