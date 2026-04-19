# Detection Playbook: Identity-Based Intrusion (Scattered Spider)

---

## Objective

Detect and investigate identity-driven attacks involving social engineering, MFA manipulation, and lateral movement using legitimate tools.

---

## Threat Summary

Attackers impersonate IT support to trick users into initiating remote sessions and approving authentication changes. They then hijack identities and move laterally using valid credentials.

---

## Key Detection Signals

* Remote support tool execution by non-IT users
* New MFA enrollment or authentication change
* Login from new IP/device shortly after MFA change
* Same account accessing multiple systems/services rapidly
* Access to sensitive resources outside user baseline

---

## Detection Logic (Correlation)

### High Confidence Scenario

1. Remote support session initiated
2. MFA method modified
3. New login from unfamiliar IP

→ Indicates likely account takeover via social engineering

---

# Detection Rules (Sigma)

---

## 1. Remote Support Tool Execution (Non-IT Users)

```yaml
title: Suspicious Remote Support Tool Execution by Non-IT User
id: scattered-spider-remote-tool
status: experimental
logsource:
  product: windows
  category: process_creation
detection:
  selection:
    Image|endswith:
      - quickassist.exe
      - screenconnect.clientservice.exe
      - netsupport.exe
  filter_it_users:
    User|contains:
      - IT
      - admin
  condition: selection and not filter_it_users
fields:
  - Image
  - User
  - CommandLine
falsepositives:
  - Legitimate remote support usage
level: medium
```

---

## 2. MFA Authentication Method Change

```yaml
title: MFA Method Change Detected
id: scattered-spider-mfa-change
status: experimental
logsource:
  product: azure
  service: auditlogs
detection:
  selection:
    OperationName:
      - "Update user"
      - "Add authentication method"
  keywords:
    - MFA
    - authentication
  condition: selection
fields:
  - UserId
  - OperationName
falsepositives:
  - Legitimate MFA reset
level: medium
```

---

## 3. Suspicious Login from New Location

```yaml
title: Suspicious Login from New Location
id: scattered-spider-new-login
status: experimental
logsource:
  product: azure
  service: signinlogs
detection:
  selection:
    ResultType: 0
  condition: selection
fields:
  - UserPrincipalName
  - IPAddress
  - Location
falsepositives:
  - VPN usage
level: low
```

---

## 4. Correlated Identity Takeover (High Confidence)

```yaml
title: Possible Account Takeover via MFA Change + New Login
id: scattered-spider-correlation
status: experimental
description: Detects MFA change followed by login from new IP
logsource:
  product: azure
detection:
  mfa_change:
    OperationName: "Add authentication method"
  login:
    ResultType: 0
  timeframe: 10m
  condition: mfa_change followed_by login
level: high
```

---

# 🧠 Investigation Workflow

## Step 1 — Validate Remote Session

* Check process execution:

  * Quick Assist / ScreenConnect / NetSupport
* Confirm user role (non-IT = suspicious)

---

## Step 2 — Review Identity Changes

* MFA enrollment events
* Authentication method changes

---

## Step 3 — Analyse Login Behaviour

* New IP / geolocation
* Device anomalies
* Timing after MFA change

---

## Step 4 — Trace Lateral Movement

* Same account across multiple hosts
* Administrative actions

---

## Step 5 — Assess Data Access

* Sensitive file access
* Unusual downloads

---

# Response Actions

* Disable compromised account
* Revoke sessions and tokens
* Remove unauthorized MFA devices
* Block suspicious IPs
* Investigate blast radius

---

# Detection Gaps

* Legitimate tools blending into normal activity
* Lack of MFA monitoring
* Weak user behavioural baselining

---

# MITRE ATT&CK Mapping

* T1566 — Phishing (Vishing)
* T1078 — Valid Accounts
* T1098 — Account Manipulation
* T1021 — Remote Services

---

# Analyst Notes

This attack is difficult to detect because:

* No malware is required
* Legitimate tools are abused
* Identity systems are the primary attack surface
---
