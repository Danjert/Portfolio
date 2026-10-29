# Corridor - Web Application IDOR & Cryptographic Failure Assessment

## Overview
This directory contains a multi-perspective security assessment of the **Corridor** web application environment. The scenario evaluates the integrity of resource identifiers, input validation mechanisms, session authorization controls, and the risks associated with weak, reversible cryptographic hashing algorithms.

## Repository Structure
- [Executive Report & GRC Mapping (Executive_report.md)](./Executive_report.md) Strategic business impact briefing, NIST CSF/SP 800-53 control mappings, MITRE ATT&CK TTP alignment, and OWASP Top 10:2025 risk integration for leadership and GRC teams.
- [Technical Walkthrough & Verification (Corridor.md)](./Corridor.md) Step-by-step technical methodology, commands, terminal output, and flag verification steps for SOC and penetration testing teams.

## Key Takeaways & Skills Demonstrated
- **Web Application Security:** Identification and exploitation of Insecure Direct Object Reference (IDOR) vulnerabilities via missing server-side authorization checks.
- **Enumeration & Cryptographic Analysis:** Endpoint discovery, cryptographic primitive identification, hash reversal, and offline brute-force analysis.
- **Framework Alignment:** Practical mapping across NIST standards, MITRE ATT&CK, and the OWASP Top 10:2025.
