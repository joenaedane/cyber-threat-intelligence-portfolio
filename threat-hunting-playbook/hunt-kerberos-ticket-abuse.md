# Threat Hunt: Kerberos Ticket Abuse

---

## Objective

Identify abnormal Kerberos authentication behaviours that may indicate credential access, lateral movement preparation, privilege escalation, or persistence activity within an Active Directory environment.

---

## Hunting Hypothesis

An attacker who has compromised a domain account may abuse Kerberos ticketing mechanisms to:

* Request large volumes of service tickets for offline password cracking (Kerberoasting)
* Impersonate privileged users via delegation abuse
* Forge authentication tickets for persistence (Golden Ticket)
* Move laterally using stolen or forged Kerberos tickets

Such activity generates authentication patterns that deviate from normal user and service behaviour.

---

## Data Sources

Primary Telemetry:

* Windows Security Log (Domain Controller)

Key Events:

* **4768** — Kerberos Authentication Ticket (TGT) Requested
* **4769** — Kerberos Service Ticket (TGS) Requested
* **4624** — Successful Logon
* **4672** — Special Privileges Assigned
* **4741** — Computer Account Created
* **5136** — Directory Attribute Modified

Optional Enrichment:

* Sysmon Event ID 1 (Process Creation)
* EDR telemetry (LSASS access / suspicious tools)

---

## Hunt Scenario 1 — Kerberoasting Behaviour

### Behavioural Indicators

* High volume of Event 4769 within short timeframe
* Requests targeting multiple unique SPNs
* RC4-HMAC encryption type usage
* Requests from single compromised workstation
* Activity outside normal working hours
* Non-privileged user requesting tickets for privileged service accounts

### Example Hunt Logic

* Group Event 4769 by User + Host
* Identify bursts exceeding behavioural baseline
* Count distinct Service Names requested

### Investigation Steps

1. Identify originating workstation.
2. Correlate with Event 4624 timeline.
3. Review endpoint telemetry for credential access tools.
4. Determine privilege level of targeted service accounts.
5. Assess likelihood of offline password cracking attempt.

---

## Hunt Scenario 2 — Delegation Abuse Behaviour

### Behavioural Indicators

* Creation of unexpected machine accounts (Event 4741)
* Modification of delegation-related attributes (Event 5136)
* Sudden service ticket requests impersonating privileged users
* Access to administrative shares from non-admin hosts
* Abnormal S4U ticket request patterns

### Investigation Steps

1. Identify new computer objects created recently.
2. Review directory attribute change logs.
3. Trace Kerberos ticket usage following attribute modification.
4. Confirm whether impersonation of privileged users occurred.
5. Investigate lateral movement attempts toward Domain Controllers.

---

## Hunt Scenario 3 — Golden Ticket / Forged Ticket Usage

### Behavioural Indicators

* Kerberos logons without preceding Event 4768
* Authentication activity from unusual hosts
* Extremely long ticket lifetime anomalies
* Privileged actions without standard authentication sequence
* Sudden domain-wide administrative access patterns

### Investigation Steps

1. Validate KRBTGT account exposure indicators.
2. Build authentication timeline for suspected user.
3. Compare ticket lifetime against domain policy.
4. Identify persistence patterns across multiple hosts.
5. Assess domain dominance likelihood.

---

## Detection Blind Spots

* Low-and-slow ticket requests evading volume-based detection
* AES-encrypted Kerberos tickets reducing RC4 hunting visibility
* Limited domain controller log retention
* Attackers operating from trusted administrative hosts
* Automation or service accounts masking abnormal behaviour
* Forged tickets mimicking legitimate authentication flows

---

## Threat Intelligence Context

Kerberos abuse techniques are commonly observed in:

* Ransomware pre-domain-compromise staging
* Post-phishing credential access campaigns
* Identity persistence operations following domain breach
* Advanced lateral movement during privilege escalation

These behaviours represent **identity-centric attack tradecraft**, where adversaries exploit trust relationships rather than software vulnerabilities.

---

## Advanced Detection Strategy: Baseline Deviation Hunting

Low-volume Kerberos abuse attempts may evade traditional threshold-based detection.

Behavioural detection should focus on identity baseline deviation, including:

* Service ticket requests from previously unseen hosts
* Gradual increase in unique SPNs accessed by a user
* Authentication activity outside normal time windows
* Ticket requests followed by lateral movement indicators
* Changes in privilege usage patterns

Effective detection requires correlation of authentication telemetry with endpoint and network activity to identify behavioural clusters indicative of staged identity attacks.


## ATT&CK Mapping

* T1558 — Steal or Forge Kerberos Tickets
* T1558.003 — Kerberoasting
* T1558.001 — Golden Ticket
* T1134 — Access Token Manipulation
* T1078 — Valid Accounts

---

## Key Lessons Learned

* Kerberos abuse often manifests as behavioural anomalies rather than signature indicators.
* Attack graph tooling must be complemented by host-level validation.
* Persistent privilege exposure (e.g., delegation abuse) is higher risk than transient session-based paths.
* Effective detection requires correlation of authentication, directory, and endpoint telemetry.

---
