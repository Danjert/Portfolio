# Corridor - Web Path Enumeration & Cryptographic Hash Analysis

## Briefing

According to OWASP Cheat Sheet Series Insecure Direct Object Reference (IDOR) is a vulnerability that arises when attackers can access or modify objects by manipulating identifiers used in a web application's URLs or parameters. It occurs due to missing access control checks, which fail to verify whether a user should be allowed to access specific data.

---

## Metadata & Environment
* **Target Environment:** TryHackMe Corridor Room (Lab IP: `10.67.177.23`, for this session)
* **Attacker Host:** Kali Linux (VirtualBox)
* **Tools:** OpenVPN, Firefox, Name That Hash (nth), John the Ripper, md5sum

---

## 1. Reconnaissance & Staging

### VPN Connection & Reachability
* **Objective:** Establish a secure connection to the TryHackMe private network and verify routing to the target lab machine.
* **Execution:**
  * Verified local OpenVPN installation:
    ```bash
    openvpn --version
    ```
    ![OpenVpn Version](../Assets/Corridor1.png)

  * Established the tunnel using the downloaded configuration:
    ```bash
    sudo openvpn filename.ovpn
    ```
  * Verified connectivity via ping:
    ```bash
    ping -c 3 10.67.177.23
    ```
  * **Result:** Tunnel successfully initialized and ping responses confirmed host reachability.

---

## 2. Enumeration & Application Analysis

* **Initial Discovery:** Navigating to `http://10.67.177.23/` via Firefox reveals an interface displaying a corridor with 13 doors.

  ![Corridor Initial View](../Assets/Corridor2.png)

* **Observation:** Clicking on the doors routes me to empty rooms, each containing a unique identifier in the URL path. For example:
  * First door to the left: `http://10.67.177.23/c4ca4238a0b923820dcc509a6f75849b`
  * Middle door: `http://10.67.177.23/8f14e45fceea167a5a36dedd4bea2543`
* **Analysis:** The URL parameters look very similar to cryptographic hash strings rather than standard sequential integer IDs.

---

## 3. Cryptographic Analysis & Hash Identification

**Recognition:** To determine the hashing algorithm used in the URL paths, the `Name That Hash` (`nth`) utility was utilized:
```bash
nth -t c4ca4238a0b923820dcc509a6f75849b
```

**Finding:** `nth` listed `MD5` as the most likely algorithm for this string. The identification was confirmed in the next step, when John the Ripper cracked the values using `--format=raw-md5`.

**Processing:** A text file `hash.txt` was created containing the extracted hashes for each visible door.

`John the Ripper` was used against the MD5 hashes to map the strings back to their plaintext inputs:

```bash
john --format=raw-md5 hash.txt
```

**Results inspection:**

```bash
john --show --format=raw-md5 hash.txt
```

**Finding:** The hashes correspond directly to sequential integer values (representing door numbers 1 through 13).

![Corridor Layout](../Assets/Corridor3.png)

---

## 4. Exploitation & Flag Verification

* **Hypothesis:** Given that the visible doors map to sequential integers (1–13), the challenge objective ("Can you find your way back to where you came?") implies the existence of an origin (integer `0`) or an adjacent path (`14`).
* **Payload Generation:** Computed the MD5 hashes for integers `0` and `14` using the terminal:
  ```bash
  # Generate hash for 14
  echo -n "14" | md5sum   # aab3238922bcc25a6f606eb525ffdc56

  # Generate hash for 0
  echo -n "0" | md5sum    # cfcd208495d565ef66e7dff9f98764da
  ```

**Testing Both Paths:**

Testing path **14** `aab3238922bcc25a6f606eb525ffdc56`: Returned a not found state.

![not Found](../Assets/Corridor4.png)

Testing path **0** `cfcd208495d565ef66e7dff9f98764da`: Successfully resolved and revealed the `flag`.

![Flag](../Assets/Corridor5.png)

---

## 5. Findings Summary

| Item | Result |
| :--- | :--- |
| Identifiers are unsalted MD5 hashes of the integers `1` to `13` | Confirmed (cracked offline with John the Ripper) |
| Door `0` (`cfcd208495d565ef66e7dff9f98764da`) is reachable without a session or credentials | Confirmed, the flag was revealed |
| Door `14` (`aab3238922bcc25a6f606eb525ffdc56`) | Not found |

Impact, framework mapping and remediation are covered in the [Executive Report](./Executive_Report.md).

## 6. Scope & Limitations
* Performed only in the TryHackMe lab environment. No production systems were tested.
* Any production impact is described as hypothetical in the executive report.
* The flag value is intentionally not reproduced in this document.
