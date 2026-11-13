# Executive Security Report: TakeOver Analysis
**Target Asset:** TryHackMe – TakeOver Room (`futurevera.thm`)  
**Assessment Type:** External Reconnaissance, Virtual Host Enumeration & Misconfiguration Review  

> **Note:** This report blends hands-on lab activity with realistic attacker scenarios. Items marked **(Lab)** were performed in the lab. Items marked **(Hypothetical)** describe what a real attacker could do with the finding.

---

## 1. Business Impact Briefing (Non-Technical Summary)

### Current Situation
During a security review of the `futurevera.thm` perimeter, a company web address was found still pointing to an external cloud storage service (Amazon S3) that no longer appeared to be in use. The DNS record was never removed when the cloud resource was retired.

### Business Impact
* **Brand Hijacking & Impersonation:** **(Hypothetical)** An attacker could claim the abandoned cloud resource and serve their own content under the company's official domain name. This creates risk of phishing, fraud, and reputational harm, because visitors would trust the address.
* **Information Exposure:** Forgotten subdomains can reveal internal naming, assets, or content that were never meant to be public.

### Recommended Actions
1. **Remove the Orphaned DNS Record:** Delete the record pointing to the unmanaged cloud resource.
2. **Standardize Offboarding:** Require IT to remove or update the related DNS records whenever a cloud service or vendor contract is retired, and to confirm no other assets still depend on it.

---

## Executive Findings Overview

| Severity | Finding |Impact | Fix |
| :--- | :--- | :--- | :--- |
| **High** | **Subdomain Takeover via Orphaned Cloud Binding** | Allows unauthorized third parties to claim abandoned cloud resources (such as AWS S3) linked via dangling DNS, enabling brand impersonation and credential phishing. | Delete orphaned DNS CNAME records immediately upon retiring cloud assets and enforce strict decommissioning offboarding checklists. |
| **Medium** | **Unrestricted Subdomain Enumeration & Asset Exposure** | Exposes hidden internal service paths, portals, and infrastructure metadata. | Implement continuous External Attack Surface Management (EASM) and restrict internal-facing subdomains behind perimeter access controls. |

---

## 2. Executive Summary
As organizations expand across cloud providers and third-party platforms, lingering DNS entries are a common attack vector. This assessment examined the `futurevera.thm` domain to identify exposed web services, asset visibility gaps, and misconfigurations.

Through virtual host discovery and TLS certificate analysis **(Lab)**, the assessment found a subdomain whose DNS record routes to an external AWS S3 endpoint. **(Hypothetical)** If the bucket were unclaimed, an attacker could register it and control the content served on the company's subdomain. The bucket was not claimed in the lab.

---

## 3. Cyber Kill Chain® Alignment

*Scope note: This assessment blends lab activity with realistic attacker scenarios. Each phase is tagged **[Performed in lab]**, **[Partially performed]**, or **[Hypothetical]**.*

* **Reconnaissance [Performed in lab]:** Ran an Nmap port scan (SSH, HTTP, HTTPS), enumerated virtual hosts with Gobuster (`portal.`, `blog.`, `support.`), and inspected certificate Subject Alternative Names, which revealed `secrethelpdesk********.support.futurevera.thm`.
* **Weaponization [Hypothetical]:** An attacker would claim the unclaimed S3 bucket and stage malicious content, such as a credential-harvesting page.
* **Delivery [Hypothetical]:** The attacker would send victims links to the trusted subdomain, for example via phishing, so the content appears legitimate.
* **Exploitation [Partially performed]:** Confirmed that the subdomain points to an S3 endpoint that could be claimed. The bucket itself was not claimed as part of the lab.

---

## 4. MITRE ATT&CK® Framework Mapping

To contextualize the findings for SOC and Threat Intelligence teams:

| Tactic | Technique | Sub-technique | Description & Context |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | T1595: Active Scanning | N/A | **(Lab)** Ran an Nmap port scan to identify open services (SSH, HTTP, HTTPS). |
| **Reconnaissance** | T1590: Gather Victim Network Information | T1590.002: DNS | **(Lab)** Examined DNS relationships across the primary domain and its subdomains. |
| **Reconnaissance** | T1595: Active Scanning | T1595.003: Wordlist Scanning | **(Lab)** Enumerated virtual hosts with `Gobuster` and discovery wordlists to find the `portal.`, `blog.`, and `support.` subdomains. |
| **Reconnaissance** | T1596: Search Open Technical Databases | T1596.003: Digital Certificates | **(Lab)** Reviewed TLS certificate Subject Alternative Names to uncover `secrethelpdesk934752.support.futurevera.thm`. |
| **Resource Development** | T1584: Compromise Infrastructure | T1584.001: Domains | **(Hypothetical)** The orphaned CNAME pointing to an unclaimed AWS S3 bucket would allow an attacker to take control of the subdomain's content and traffic. The bucket was not claimed in the lab. |

---

## 5. NIST CSF & NIST SP 800-53 Control Alignment

* **NIST CSF 2.0: Identify (ID) / Asset Management (ID.AM):**
  * **Control Gap:** Incomplete visibility and lifecycle tracking of external digital assets and cloud bindings.
  * **NIST SP 800-53 Mapping:** **CM-8 (System Component Inventory)**: maintain an accurate inventory of public-facing assets, subdomains, and cloud resources, including DNS records that reference them.
* **NIST CSF 2.0: Govern (GV) / Supply Chain Risk Management (GV.SC):**
  * **Control Gap:** No enforced offboarding process to remove DNS references when a third-party or cloud service is retired.
  * **NIST SP 800-53 Mapping:** **SA-9 (External System Services)**: manage and monitor the use of external services, including their retirement.

---

## 6. Strategic Recommendations
1. **Deploy Continuous External Attack Surface Management (EASM):** Automate scanning of public DNS records and third-party cloud integrations to detect dangling or unmanaged assets.
2. **Enforce Decommissioning Protocols:** Use cross-functional checklists that pair every cloud resource deletion with removal of its DNS records.
3. **Monitor Certificate Transparency Logs:** Regularly review public CT logs for unexpected Subject Alternative Names tied to company domains.

---

## 7. Framework References & Resources

### Attack Lifecycle
* [Lockheed Martin Cyber Kill Chain® Overview](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)

### Threat Intelligence & TTP Mapping
* [MITRE ATT&CK® Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/)
* [T1595: Active Scanning](https://attack.mitre.org/techniques/T1595/)
* [T1590.002: Gather Victim Network Information: DNS](https://attack.mitre.org/techniques/T1590/002/)
* [T1595.003: Active Scanning: Wordlist Scanning](https://attack.mitre.org/techniques/T1595/003/)
* [T1596.003: Search Open Technical Databases: Digital Certificates](https://attack.mitre.org/techniques/T1596/003/)
* [T1584.001: Compromise Infrastructure: Domains](https://attack.mitre.org/techniques/T1584/001/)

### Governance, Risk, & Compliance
* [NIST Cybersecurity Framework (CSF 2.0)](https://www.nist.gov/cyberframework)
* [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
* [NIST SP 800-53 Control CM-8](https://csf.tools/reference/nist-sp-800-53/r5/cm-8/)
