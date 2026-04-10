# Incident Report: Domain Control via Redundant Service Account Privilege Abuse on Domain Controller (DC01)

---

## Source of Analysis

This incident assessment is derived from behavioural observations recorded during a simulated attack path analysis exercise conducted within a controlled Active Directory lab environment.

Technical Simulation Reference:
`lab-environment/presistence-c2/domain-account-redunancy.md`

---

## 1. Incident Overview

Suspicious account creation and privileged group modifications were observed on the Domain Controller (DC01), indicating the establishment of redundant administrative access. Investigation confirmed that the attacker created an additional service account (`svc_ops`) and elevated it to Domain Administrator privileges alongside an existing privileged account (`svc_backup`).

This activity demonstrates a transition from single-point persistence to **durable domain control**, where multiple independent access paths are established to ensure continued attacker presence.

The incident is classified as Critical severity due to the attacker’s ability to maintain control even after partial remediation efforts.

---

## 2. Initial Detection

The incident was identified through abnormal account management activity and privileged group modifications on DC01.

Key indicators:

* Creation of a new domain account (`svc_ops`)
* Addition of account to Domain Admins group
* Privileged logon activity associated with newly created account
* Multiple service accounts holding Domain Administrator privileges

Relevant telemetry sources:

* Windows Security Logs (Event IDs: 4720, 4728, 4624, 4672)
* Account and group management logs

---

## 3. Investigation Findings

### 3.1 Redundant Persistence Mechanism

* Target Host: DC01
* Accounts Involved: `svc_backup`, `svc_ops`
* Technique: Account creation and privilege escalation

Observed behavior:

* New account (`svc_ops`) created (Event ID 4720)
* Account added to Domain Admins group (Event ID 4728)
* Multiple accounts now possess Domain Administrator privileges

The attacker established redundancy by introducing multiple privileged identities, ensuring continued access even if one account is removed.

---

### 3.2 Privileged Access Validation

* Accounts Used: `corp\svc_backup`, `corp\svc_ops`

Observed behavior:

* Successful authentication using newly created account
* Assignment of privileged tokens (Event ID 4672)
* Independent administrative access confirmed

This demonstrates that access is no longer dependent on a single compromised identity.

---

### 3.3 Resilience Against Remediation

Observed behavior:

* Removal of one privileged account does not eliminate attacker access
* Secondary account maintains Domain Admin privileges
* Continued administrative control despite partial remediation

This indicates a shift from persistence to **resilient domain control**, significantly increasing attacker survivability.

---

### 3.4 Identity Layer Compromise

Observed behavior:

* Multiple privileged accounts controlled by attacker
* Abuse of legitimate identity constructs within Active Directory
* No reliance on malware or external tools

This confirms compromise of the **identity layer**, which represents the highest level of control within the domain.

---

### 3.5 Timeline

| Time     | Event                                          |
| -------- | ---------------------------------------------- |
| 09:45:36 | svc_ops account created (Event ID 4720)        |
| 09:47:21 | svc_ops added to Domain Admins (Event ID 4728) |
| 09:50:14 | svc_ops authenticated to DC01 (Event ID 4624)  |
| 09:50:14 | Privileged access assigned (Event ID 4672)     |

---

## 4. Evidence and IOCs

* Accounts:

  * svc_backup
  * svc_ops

* Privileged Group:

  * Domain Admins

* Indicators:

  * Multiple service accounts with Domain Admin privileges
  * Newly created administrative accounts
  * Privileged logon activity from atypical accounts

---

## 5. Impact Assessment

* Systems affected:

  * DC01 (Domain Controller)

* Accounts involved:

  * svc_backup
  * svc_ops

* Risk Level: Critical

  * Multiple persistent administrative access paths established
  * High resistance to remediation efforts
  * Full control over Active Directory identity infrastructure

---

## 6. Containment Actions

* Removed unauthorized accounts from Domain Admins group
* Disabled or deleted malicious service accounts
* Reviewed all privileged group memberships
* Enforced stricter controls on account creation and privilege escalation
* Monitored for re-establishment of unauthorized privileged accounts

---

## 7. Root Cause

* Lack of monitoring for account creation and privilege escalation
* Excessive privileges assigned without validation
* Absence of alerting on multiple privileged identities
* Weak governance over service account usage

---

## 8. MITRE ATT&CK Mapping

| Technique            | ID     |
| -------------------- | ------ |
| Account Manipulation | T1098  |
| Valid Accounts       | T1078  |
| Create Account       | T1136  |
| Privilege Escalation | TA0004 |

---

## 9. Conclusion

The attacker successfully transitioned from persistence to **durable domain control** by establishing multiple privileged service accounts within Active Directory.

By creating and elevating an additional account (`svc_ops`) alongside an existing backdoor (`svc_backup`), the attacker ensured continued access even in the event of partial remediation.

This approach significantly increases attacker resilience and demonstrates how identity-based techniques can be used to maintain long-term control without reliance on malware or external tools.

The incident highlights the critical importance of monitoring account creation, privileged group modifications, and anomalous authentication patterns to detect and prevent identity-based attacks within enterprise environments.
