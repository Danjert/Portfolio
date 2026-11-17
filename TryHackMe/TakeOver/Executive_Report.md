# Executive Security Report: TakeOver Analysis
**Target Asset:** TryHackMe – TakeOver Room (`futurevera.thm`)  
**Assessment Type:** External Reconnaissance, Virtual Host Enumeration & Misconfiguration Review  

| Field | Value |
| :--- | :--- |
| **Author** | SOC Analyst |
| **Date** | 10/26/2026 |
| **Version** | 1.0 |
| **Scope** | TryHackMe TakeOver room (lab environment) |
| **Audience** | Leadership, GRC and security operations |
| **Technical detail** | [TakeOver.md](./TakeOver.md) |

> **Note:** This report combines hands-on lab activity with realistic attacker scenarios. Items marked **(Lab)** were performed in the lab. Items marked **(Hypothetical)** describe what a real attacker could do with the finding.

---

## 1. Business Impact Briefing

### Current Situation
During a security review of the `futurevera.thm` perimeter, a hidden company web address (disclosed in the TLS certificate of a support page) redirected visitors to an external Amazon S3 website endpoint. This is the typical symptom of a retired cloud resource whose DNS entry was never removed. The lab could not confirm the underlying configuration, because name resolution was done through a local hosts file, and it did not test whether the bucket could be claimed.

### Business Impact
* **Brand Hijacking & Impersonation:** **(Hypothetical)** An attacker could claim the abandoned cloud resource and serve their own content under the company's official domain name. This creates risk of phishing, fraud, and reputational harm, because visitors would trust the address.
* **Information Exposure:** Forgotten subdomains can reveal internal naming, assets, or content that were never meant to be public.

### Recommended Actions
1. **Review and Remove the Orphaned Record:** Check how this hostname is configured (DNS record or redirect). If the cloud resource is no longer in use, delete the record or redirect.
2. **Standardize Offboarding:** Require IT to remove or update the related DNS records whenever a cloud service or vendor contract is retired, and to confirm no other assets still depend on it.

---

## 2. Executive Summary
As organizations expand across cloud providers and third-party platforms, lingering DNS entries are a common attack vector. This assessment examined the `futurevera.thm` domain to identify exposed web services, asset visibility gaps, and misconfigurations.

Through virtual host discovery and TLS certificate analysis **(Lab)**, the assessment found a hidden subdomain, listed in a certificate's Subject Alternative Names, that redirects to an external AWS S3 website endpoint. **(Hypothetical)** If the bucket were unclaimed, an attacker could register it and control the content served on the company's subdomain.

---

## 3. Executive Findings Overview

| Severity | Finding | Impact | Fix |
| :--- | :--- | :--- | :--- |
| **Medium** (High if the bucket is confirmed claimable) | **Potential Subdomain Takeover via Redirect to an External S3 Endpoint** | A hidden company hostname redirects to an external S3 website endpoint. If the underlying bucket were unclaimed, a third party could serve content under the company's domain (brand impersonation, credential phishing). *Claimability was not tested.* | Verify how the hostname is configured, remove the record or redirect if the resource is retired, and enforce offboarding checklists that remove related DNS entries. |
| **Low** | **Internal Hostname Disclosure via TLS Certificate SAN** | The support site's certificate lists an unlinked hostname, revealing hidden services and naming. Virtual hosts were also discoverable with a wordlist. | Keep unpublished hostnames out of public certificates, restrict internal-facing services behind perimeter access controls, and run continuous External Attack Surface Management (EASM). |

**Rating method.** Severity reflects likelihood and impact for a production environment, so ratings are hypothetical. Finding 1 is rated Medium because the takeover condition (an unclaimed bucket) was not verified, and it would rise to High if confirmed. Evidence for each finding is in [TakeOver.md](./TakeOver.md): Phase 3 (virtual hosts), Phase 4 (certificate SAN and redirect).

---

## 4. Cyber Kill Chain® Alignment

* **Reconnaissance (Lab):** Ran an Nmap port scan (SSH, HTTP, HTTPS), enumerated virtual hosts with Gobuster (`portal.`, `blog.`, `support.`), and inspected certificate Subject Alternative Names, which revealed `secrethelpdesk******.support.futurevera.thm`.
* **Weaponization (Hypothetical):** An attacker would claim the unclaimed S3 bucket and stage malicious content, such as a credential-harvesting page.
* **Delivery (Hypothetical):** The attacker would send victims links to the trusted subdomain, for example via phishing, so the content appears legitimate.
* **Exploitation (Hypothetical):** Victims visiting the trusted subdomain would be served attacker-controlled content from the hijacked S3 bucket (e.g., capturing credentials). In the lab, only the redirect to the S3 endpoint was confirmed; active bucket takeover was out of scope.

---

## 5. MITRE ATT&CK® Framework Mapping

To contextualize the findings for SOC and Threat Intelligence teams:

| Tactic | Technique | Sub-technique | Description & Context |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | T1595: Active Scanning | N/A | **(Lab)** Ran an Nmap port scan to identify open services (SSH, HTTP, HTTPS). |
| **Reconnaissance** | T1590: Gather Victim Network Information | T1590.002: DNS | **(Lab)** Examined DNS relationships across the primary domain and its subdomains. |
| **Reconnaissance** | T1595: Active Scanning | T1595.003: Wordlist Scanning | **(Lab)** Enumerated virtual hosts with `Gobuster` and discovery wordlists to find the `portal.`, `blog.`, and `support.` subdomains. |
| **Reconnaissance** | T1596: Search Open Technical Databases | T1596.003: Digital Certificates | **(Lab)** Reviewed the TLS certificate's Subject Alternative Names to uncover `secrethelpdesk******.support.futurevera.thm`. *Note: T1596.003 normally refers to certificate transparency data. It is the closest mapping, since the certificate was read directly from the target's HTTPS service.* |
| **Resource Development** | T1584: Compromise Infrastructure | T1584.001: Domains | **(Hypothetical)** A dangling DNS record pointing to an unclaimed AWS S3 bucket would allow an attacker to take control of the subdomain's content and traffic. |

---

## 6. NIST CSF & NIST SP 800-53 Control Alignment

* **NIST CSF 2.0: Identify (ID) / Asset Management (ID.AM):**
  * **Control Gap:** Incomplete visibility and lifecycle tracking of external digital assets and cloud bindings.
  * **NIST SP 800-53 Mapping:** **CM-8 (System Component Inventory)**: maintain an accurate inventory of public-facing assets, subdomains, and cloud resources, including DNS records that reference them.
* **NIST CSF 2.0: Govern (GV) / Supply Chain Risk Management (GV.SC):**
  * **Control Gap:** No enforced offboarding process to remove DNS references when a third-party or cloud service is retired.
  * **NIST SP 800-53 Mapping:** **SA-9 (External System Services)**: manage and monitor the use of external services, including their retirement.

---

## 7. Strategic Recommendations
1. **Deploy Continuous External Attack Surface Management (EASM):** Automate scanning of public DNS records and third-party cloud integrations to detect dangling or unmanaged assets.
2. **Enforce Decommissioning Protocols:** Use cross-functional checklists that pair every cloud resource deletion with removal of its DNS records.
3. **Monitor Certificate Transparency Logs:** Regularly review public CT logs for unexpected Subject Alternative Names tied to company domains.

---

### How to Verify the Fixes
* Confirm that the hidden hostname no longer resolves or redirects to the external endpoint, and that no other DNS record references a retired cloud resource.
* Re-run virtual host discovery and certificate review after the change, and compare against the asset inventory.
* Add a recurring check that flags DNS entries pointing to cloud endpoints that return "not found" or unclaimed-resource errors.

---

## 8. Limitations
* The assessment was a single run in a lab environment, and name resolution used a local hosts file.
* Bucket claimability was not tested, and no bucket was registered, so the takeover impact is hypothetical.
* Severity ratings are estimates based on the stated rating method.

---

## 9. Framework References & Resources

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
