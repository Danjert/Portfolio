# Executive Security Report: Corridor Web Application Analysis

| Field | Value |
| :--- | :--- |
| **Author** | SOC Analyst |
| **Date** | 10/20/2026 |
| **Version** | 1.0 |
| **Target** | TryHackMe Corridor room (lab environment) |
| **Audience** | Leadership, GRC and application owners |
| **Technical detail** | [Corridor.md](./Corridor.md) |

> **Note:** This report is based on a hands-on lab (TryHackMe – Corridor). Findings marked **(Lab)** were performed in the lab environment. Statements marked **(Hypothetical)** describe how the same weakness could play out in a real production environment.

## Executive Summary
A security assessment was performed on the **Corridor** web application to evaluate the integrity of resource identifiers, input validation, and server-side authorization. The assessment uncovered two connected design flaws: predictable resource identifiers derived from an unsalted MD5 hash of sequential integers, and no server-side authorization on the endpoints those identifiers protect. Together, these allowed an unauthenticated actor to reach resources that were never meant to be exposed.

In a production environment, this pattern (broken access control through predictable direct object references, commonly called IDOR) could expose other users' data or administrative functions to anyone able to guess or fabricate an identifier.

**Recommended action:** enforce server-side authorization on every route. Replacing the identifiers alone would not fix the problem.

---

## Executive Findings Overview

| Severity | Finding |Impact | Fix |
| :--- | :--- | :--- | :--- |
| **High** | **Absence of Server-Side Authorization (IDOR)** | Allows unauthenticated actors to bypass access controls and reach confidential endpoints or protected assets. | Implement centralized, server-side session and role-based validation (`RBAC`) on every route. |
| **Medium** | **Predictable Resource Identifiers & Weak Hashing** | Allows all valid URLs to be discovered by generating predictable MD5 hashes from sequential values (security through obscurity).| Replace sequential values and deprecated hashes with unguessable random tokens (such as `UUIDv4`). |

**Rating method.** Severity reflects likelihood and impact if the same flaw existed in a production application that handles sensitive data, so the ratings are hypothetical. For reference, Finding 1 would score about **7.5 (High)** under CVSS v3.1 with the vector `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`. Finding 2 is rated Medium because it is a contributing weakness that is not independently exploitable.

---

## Scope & Framework Mapping
* **Assessment Objective:** Evaluate web application path security, identifier predictability, and access control enforcement.
* **Core Methodologies:** Manual path probing, hash identification, offline hash reversal, and boundary testing.
* **Standards & Framework Alignment:**
  * **NIST CSF 2.0:** Protect (PR.AA – Identity Management, Authentication, and Access Control; PR.DS – Data Security)
  * **NIST SP 800-53 Rev. 5:** AC-3 (Access Enforcement), SC-13 (Cryptographic Protection), SI-10 (Information Input Validation)
  * **MITRE ATT&CK:** T1595 (Active Scanning), T1190 (Exploit Public-Facing Application), T1110.002 (Brute Force: Password Cracking, related to offline hash reversal), T1078 (Valid Accounts, hypothetical)
  * **OWASP Top 10 (2025):** A01:2025 – Broken Access Control, A04:2025 – Cryptographic Failures

---

## Detailed Findings & Risk Analysis

### 1. Absence of Server-Side Authorization (OWASP A01, NIST AC-3)
* **Severity:** High
* **Vulnerability Description (Lab):** The application served content to any request whose path resolved to a valid entry point. It did not check session, user role, or context. This allowed access to an unlinked resource (door `0`) that held the lab's protected asset (the flag).
* **Business & Operational Impact:** Without server-side checks, anyone who can reach a URL can use it. **(Hypothetical)** In a real application, the same flaw could expose customer records, internal tools, or administrative pathways, directly affecting confidentiality and integrity.
* **Evidence:** [Corridor.md](./Corridor.md), section 4 (path `0` resolved with no session or credentials).

> **Note:** This is the root cause. Even a perfectly random identifier would only slow an attacker down if the server still performs no authorization check.

### 2. Predictable Resource Identifiers & Weak Hashing (OWASP A04, NIST SC-13)
* **Severity:** Medium
* **Vulnerability Description (Lab):** The application used 32-character hexadecimal strings in URL paths to represent room identifiers. Analysis showed these were unsalted **MD5 hashes** of sequential integers (`1` through `13`).
* **Business & Operational Impact:** Hashing a small, sequential set of integers does not hide them. An attacker can compute every value in seconds and enumerate the whole namespace. This is **security through obscurity**, not authorization, and it gives a false sense of protection.
* **Evidence:** [Corridor.md](./Corridor.md), section 3 (hash identification and offline reversal).

---

## Detailed NIST Framework Mapping & Control Analysis

### 1. NIST SP 800-53 Rev. 5 Control Analysis

* **AC-3 (Access Enforcement):**
  * **Requirement:** The system must enforce approved authorizations for logical access to information and system resources.
  * **Observation:** The application performed no authorization check on path parameters, so any actor who fabricated a valid value could reach the resource.
  * **Remediation:** Implement centralized, server-side session and role validation (for example, RBAC) so every request verifies user privileges before returning data.

* **SC-13 (Cryptographic Protection):**
  * **Requirement:** Cryptography must be used in accordance with applicable standards and guidelines.
  * **Observation:** MD5, a deprecated algorithm, was used to obscure sequential identifiers. Hashing is not an access control mechanism, so it was the wrong tool for this purpose.
  * **Remediation:** Stop using hashes to conceal resource references. Where identifiers must be exposed to clients, use unguessable random values (such as UUIDv4), and treat these as a supplement to authorization, not a replacement for it.

* **SI-10 (Information Input Validation):**
  * **Requirement:** The system must check the validity of inputs where malformed values could introduce vulnerabilities.
  * **Observation:** The application accepted hexadecimal path values directly into routing logic without validating them against the requesting user's context.
  * **Remediation:** Apply strict type, format, and boundary validation across all routing parameters.

### 2. NIST CSF 2.0 Function Alignment

* **Protect (PR):**
  * **PR.AA (Identity Management, Authentication, and Access Control):** The finding shows a failure to enforce access permissions on dynamic routing endpoints.
  * **PR.DS (Data Security):** Weak identifier protection combined with missing authorization led to unauthorized disclosure of a protected asset (the flag).

---

## Threat Classification & Framework Mapping

### 1. MITRE ATT&CK® Framework Alignment

The table below maps lab activity to MITRE ATT&CK for SOC and Threat Intelligence teams. Rows marked **(Lab)** were performed. Rows marked **(Hypothetical)** describe likely real-world follow-on activity and were not performed.

| Tactic | Technique | Sub-technique | Description & Context |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | T1595: Active Scanning | N/A | **(Lab)** Probed the target with ping and manual path testing to identify reachable endpoints and URL structure. |
| **Initial Access** | T1190: Exploit Public-Facing Application | N/A | **(Lab)** Accessed unlinked resources, including door `0`, by exploiting missing server-side authorization on predictable identifiers (IDOR). |
| **Credential Access** | T1110: Brute Force | T1110.002: Password Cracking | **(Lab)** Used offline hash identification and cracking (`Name That Hash`, `John the Ripper`) to map MD5 values back to the integers `1` through `13`. *Note: T1110.002 describes cracking password hashes. It is the closest mapping for offline hash reversal here, but the hashes were resource identifiers, not credentials.* |
| **Initial Access / Persistence / Privilege Escalation / Defense Evasion** | T1078: Valid Accounts | N/A | **(Hypothetical)** If the IDOR exposed user data or session material in a real application, an attacker could use it to gain valid account access. |

### 2. OWASP Top 10:2025 Risk Alignment

* **A01:2025 – Broken Access Control:**
  * **Corridor finding:** No server-side authorization on dynamic routing endpoints allowed unauthenticated access to a protected resource (the flag) via IDOR.
* **A04:2025 – Cryptographic Failures:**
  * **Corridor finding:** Predictable MD5 hashing of sequential door numbers created a false sense of security, and was reversed offline with `John the Ripper` and `Name That Hash`.

---

## Strategic Recommendations & Remediation Plan

1. **Implement Robust Access Controls (NIST AC-3), highest priority:** Enforce server-side session validation and role-based authorization on every route. Don't rely on path obfuscation to protect sensitive endpoints.
2. **Eliminate Predictable Identifiers:** Replace sequential integers and their hashes with cryptographically secure random identifiers (such as **UUIDv4**) wherever direct object references must be exposed.
3. **Retire Legacy Hashing (NIST SC-13):** Don't use MD5 and SHA-1 for security-sensitive purposes. Use modern algorithms such as SHA-256 for integrity and Bcrypt or Argon2 for password storage.
4. **Validate Inputs (NIST SI-10):** Validate and restrict all routing parameters server-side.

### How to Verify the Fixes
* Request the former door URLs (including the hash of `0`) **without** a session. The server should return `401` or `403`, and the same request with a valid, authorized session should succeed.
* Confirm that new identifiers are random and that guessing or computing values returns no valid resource.
* Add automated authorization tests to the pipeline, so permission gaps are caught.

---

## Limitations
* The assessment was a single run in a lab environment, and no production systems were tested.
* The application has no user accounts, so user-to-user data exposure was not tested. Production impact is hypothetical.
* Severity ratings are estimates based on the stated rating method.

---

## Related Resources & References
* [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
* [MITRE ATT&CK T1595: Active Scanning](https://attack.mitre.org/techniques/T1595/)
* [MITRE ATT&CK T1190: Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
* [MITRE ATT&CK T1110.002: Brute Force: Password Cracking](https://attack.mitre.org/techniques/T1110/002/)
* [MITRE ATT&CK T1078: Valid Accounts](https://attack.mitre.org/techniques/T1078/)
* [OWASP Top 10: A01:2025 – Broken Access Control](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/)
* [OWASP Top 10: A04:2025 – Cryptographic Failures](https://top10.owasp.org/2025/A04_2025-Cryptographic_Failures/)
