# Executive Security Report & Business Briefing
**Target Asset:** TryHackMe – TakeOver Room (`futurevera.thm`)  
**Assessment Type:** External Reconnaissance, Virtual Host Enumeration & Misconfiguration Review  

---

## 1. Business Impact Briefing (Non-Technical Summary)

### Current Situation
During a routine security review of `futurevera.thm`'s perimeter, an unmonitored digital asset was identified. A corporate web address was found still linking to an external cloud storage service that wasn't correctly managed during the current renovations.

### Business Impact
* **Brand Hijacking & Impersonation:** Because the web link remained active and unmonitored, an external adversary could claim the abandoned cloud endpoint and serve unauthorized or malicious content under the company's official domain name. This introduces significant risk for phishing, fraud, and reputational harm.
* **Information Exposure:** Unmonitored or forgotten subdomains can expose internal assets or confidential information to the public.

### Recommended Actions
1. **Remove the Inactive Link:** Immediately purge the DNS record pointing to the unmanaged cloud resource.
2. **Standardize Offboarding Workflows:** Enforce an enterprise policy requiring IT to audit and if necessary take the corresponding steps to assess the identified situation such as modifying the related files whenever a cloud service or vendor contract is retired.

---

## 2. Executive Summary
As organizations expand across cloud providers and third-party content platforms, unmonitored subdomains and lingering DNS entries present a common attack vector. This evaluation assessed the `futurevera.thm` domain to identify exposed web services, asset visibility gaps, and infrastructure misconfigurations susceptible to malicious actors.

Through active virtual host discovery and cryptographic certificate analysis, the assessment uncovered a DNS reference routing to an external cloud resource (`AWS S3`). This finding demonstrates how unmanaged resources can be leveraged by malicious actors to exploit the company infrastructure.

---

## 3. Cyber Kill Chain® Alignment
The assessment was structured to simulate the typical lifecycle of an attack:

* **Reconnaissance:** Performed active perimeter mapping and port discovery to analyze the attack surface and identify available network interfaces and services (SSH, HTTP, HTTPS).
* **Weaponization:** Executed targeted virtual host enumeration across web protocols using discovery wordlists to identify subdomains (`portal.`, `blog.`, `support.`).
* **Delivery:** Extracted cryptographic certificate metadata (Subject Alternative Names) from accessible web endpoints to unveil additional assets (`secrethelpdesk934752.support.futurevera.thm`).
* **Exploitation:** Followed the exposed subdomain to an unmanaged Amazon S3 bucket, validating our ability to hijack the endpoint and intercept user requests.

---

## 4. MITRE ATT&CK® Framework Mapping
To contextualize the technical findings for Security Operations Center (SOC) and Threat Intelligence teams, the observed activities align with the following Tactics, Techniques, and Sub-techniques:

| Tactic | Technique ID & Name | Sub-technique ID & Name | Description & Context |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | **T1590**: Gather Victim Network Information | **T1590.002**: Domain Properties | Mapped target perimeter, open network ports, and DNS relationships across the primary domain. |
| **Reconnaissance** | **T1592**: Gather Victim Host Information | **T1592.002**: Software Identity / Certificates | Discovered auxiliary subdomains and hidden naming structures by inspecting SSL/TLS certificate attributes. |
| **Reconnaissance** | **T1589**: Gather Victim Identity Information | **T1589.002**: Web Footprint | Identified unindexed web applications and hidden directories using targeted virtual host enumeration (`Gobuster`). |
| **Initial Access / Impact** | **T1584**: Compromise Infrastructure | **T1584.001**: Domains | Exploited an orphaned DNS CNAME record pointing to an unmanaged cloud bucket (`AWS S3`) to establish control over subdomain routing and traffic. |

---

## 5. NIST CSF & NIST SP 800-53 Control Alignment
To align findings with enterprise governance and risk compliance standards, the identified vulnerabilities correspond to control gaps under the **NIST Cybersecurity Framework (CSF)** and **NIST SP 800-53**:

* **NIST CSF Function: Identify (ID) / Asset Management (ID.AM):**
  * *Control Gap:* Incomplete visibility and lifecycle tracking of external digital assets and cloud bindings.
  * *NIST SP 800-53 Mapping:* **CM-8 (Information System Component Inventory)** – Maintaining an accurate, up-to-date inventory of all public-facing assets, subdomains, and cloud resources.
* **NIST CSF Function: Protect (PR) / Protective Technology (PR.PT):**
  * *Control Gap:* Inadequate DNS record cleanup during cloud resource offboarding.
  * *NIST SP 800-53 Mapping:* **SC-8 (Transmission Confidentiality and Integrity)** & **AC-3 (Access Enforcement)** – Enforcing strict domain verification and ensuring decommissioned cloud endpoints cannot be claimed by unauthorized entities.

---

## 6. Strategic Recommendations
1. **Deploy Continuous External Attack Surface Management (EASM):** Automate the continuous scanning of public DNS records and third-party cloud integrations to detect unmanaged assets.
2. **Enforce Decommissioning Protocols:** Establish cross-functional technical checklists requiring cloud resource deletions to be paired synchronously with DNS record removals.
3. **Audit Certificate Transparency Logs:** Regularly monitor public Certificate Transparency (CT) logs for unexpected or unlinked Subject Alternative Names (SANs) associated with root domains.

---

## 7. Framework References & Resources

### Attack Lifecycle & Methodology
* **Lockheed Martin Cyber Kill Chain®:** [Lockheed Martin Cyber Kill Chain Framework Overview](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)

### Threat Intelligence & TTP Mapping
* **MITRE ATT&CK® Matrix for Enterprise:** [Enterprise Matrix Overview](https://attack.mitre.org/matrices/enterprise/)
* **Technique T1590.002 (Gather Victim Network Information: Domain Properties):** [T1590.002 Reference](https://attack.mitre.org/techniques/T1590/002/)
* **Technique T1592.002 (Gather Victim Host Information: Software Identity / Certificates):** [T1592.002 Reference](https://attack.mitre.org/techniques/T1592/002/)
* **Technique T1589 (Gather Victim Identity Information):** [T1589 Reference](https://attack.mitre.org/techniques/T1589/)
* **Technique T1584.001 (Compromise Infrastructure: Domains):** [T1584.001 Reference & Mitigation](https://attack.mitre.org/techniques/T1584/001/)

### Governance, Risk, & Compliance (GRC)
* **NIST Cybersecurity Framework (CSF 2.0):** [NIST CSF Resource Center](https://www.nist.gov/cyberframework)
* **NIST SP 800-53 Rev. 5 (Security and Privacy Controls for Information Systems):** [NIST Computer Security Resource Center (CSRC) SP 800-53 Publication](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
* **NIST SP 800-53 Control CM-8 (System Component Inventory):** [NIST SP 800-53 Rev. 5 Control CM-8 Reference](https://csf.tools/reference/nist-sp-800-53/r5/cm-8/)