# Incident Report: Persistence via Service Account Privilege Abuse on Domain Controller (DC01)

---

## Source of Analysis
This incident assessment is derived from behavioural observations recorded during a simulated attack path analysis exercise conducted within a controlled Active Directory lab environment.

Technical Simulation Reference:  
`lab-environment/persistence-c2/domain-account-manipulation.md`

---

## 1. Incident Overview

Suspicious administrative activity was detected on the Domain Controller (DC01) involving modification of privileged group membership and subsequent use of a service account for administrative access. Investigation confirmed that the attacker established persistence by elevating an existing service account (`svc_backup`) to Domain Administrator privileges, followed by authentication using the modified account.

The incident is classified as Critical severity due to the establishment of persistent domain-level access independent of the original compromise vector.

---

## 2. Initial Detection

The incident was triggered by abnormal privilege modification and authentication behaviour observed on DC01.

Key indicators:
- Creation of account (svc_backup)
- Addition of account to privileged group (Domain Admins)
- First-time privileged logon by service account
- Administrative logon without prior baseline activity

Relevant telemetry sources:
- Windows Security Logs (Event IDs: 4728, 4624, 4672)
- Account and group management logs

---

## 3. Investigation Findings

### 3.1 Persistence Mechanism

- Target Host: DC01  
- Account Created: svc_backup  
- Technique: Account creation and privileged group membership modification  

Observed behavior:
- New domain account (`svc_backup`) created (Event ID 4720)  
- Account added to Domain Admins group (Event ID 4728)  
- Account elevated to domain-level administrative privileges  

The attacker established persistence by introducing a secondary privileged identity within Active Directory, enabling continued access independent of the original compromised account.

---

### 3.2 Privileged Access Validation

- Account Used: corp\svc_backup  
- Logon Type: Interactive / Network  

Observed behavior:
- Successful authentication using modified account  
- Assignment of privileged token (Event ID 4672)  
- Confirmation of Domain Admin group membership  

This demonstrates successful persistence independent of the original compromised identity.

---

### 3.3 Command-and-Control Behaviour

- Source Host: DC01  
- Communication Type: HTTP / ICMP  

Observed behavior:
- Repeated outbound communication to external host  
- Execution of PowerShell-based web requests or continuous ICMP traffic  
- Behaviour consistent with beaconing pattern  

This indicates establishment of a basic command-and-control (C2) channel for remote tasking.

---

### 3.4 Timeline

| Time | Event |
|------|------|
| 3/30/2026 11:43:49 | svc_backup added to Domain Admins (Event ID 4728) |
| 3/30/2026 11:48:44 | svc_backup authenticated to DC01 (Event ID 4624) |
| 3/30/2026 11:48:44 | Privileged access assigned (Event ID 4672) |
| 3/30/2026 12:01:12 | Outbound beaconing activity initiated |

---

## 4. Evidence and IOCs

The screenshots are provided in:  
`lab-environment/persistence-c2/domain-account-manipulation.md`

---

## 5. Impact Assessment

- Systems affected:
  - DC01 (compromised)

- Accounts involved:
  - svc_backup

- Risk Level: Critical  
  - Persistent domain-level access established  
  - Access independent of initial compromise path  
  - Capability for long-term control and further attacker operations  

---

## 6. Containment Actions

- Removed svc_backup from Domain Admins group  
- Disabled or reset compromised service account credentials  
- Reviewed all privileged group memberships  
- Monitored DC01 for persistence mechanisms  
- Blocked suspicious outbound communication from domain infrastructure  

---

## 7. Root Cause

- Excessive privilege exposure within Active Directory  
- Lack of monitoring for privileged group modifications  
- Absence of controls on service account usage  
- Insufficient correlation between identity changes and logon activity  

---

## 8. MITRE ATT&CK Mapping

| Technique | ID |
|----------|----|
| Account Manipulation | T1098 |
| Valid Accounts | T1078 |
| Command and Scripting Interpreter | T1059 |
| Application Layer Protocol | T1071 |

---

## 9. Conclusion

The attacker successfully established persistent domain-level access by modifying an existing service account and elevating it to Domain Administrator privileges. This allowed continued access to the environment independent of the original compromise vector.

The use of an existing account reduced detection visibility compared to account creation, while subsequent authentication confirmed successful persistence.

Additionally, outbound communication behaviour indicates the establishment of a command-and-control channel, enabling ongoing attacker operations.

This incident demonstrates how identity manipulation enables durable access and highlights the importance of monitoring privileged group changes and anomalous authentication patterns within Active Directory environments.