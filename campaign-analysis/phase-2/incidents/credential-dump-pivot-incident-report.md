# Incident Report: Credential Dumping Activity

---

## Source of Analysis
This incident assessment is derived from behavioural observations recorded during a simulated attack path analysis exercise conducted within a controlled Active Directory lab environment.

Technical Simulation Reference:
`lab-environment/credential-dumps-and-lateral-move/credential-dump-pivot.md`

## 1. Incident Overview

Suspicious activity was identified on Host-A involving potential credential access techniques. Investigation revealed that an attacker accessed sensitive system memory to extract authentication credentials, indicating credential dumping behavior. The incident is classified as High severity due to the risk of privilege escalation and lateral movement.

---

## 2. Initial Detection
The incident was triggered by detection of abnormal process activity involving LSASS access.

Key indicators:
- Process interaction with lsass.exe
- High-privilege account usage (helpdesk_admin)
- Suspicious process creation events

Relevant telemetry sources:
- Windows Security Logs (Event IDs: 4624, 4673, 4674, 4688)
- Sysmon logs (Process access events)


---

## 3. Investigation Findings

### 3.1 Initial Access

- Host: WS01  
- Initial User: corp\john  
- Access Type: Interactive login  

The attacker established a foothold on WS01 using valid domain credentials.

----

### 3.2 Privilege Escalation

The attacker identified that the account helpdesk_admin had local administrative privileges on WS01.

Actions observed:
- Elevation of privileges using administrative credentials  
- Transition to high-integrity execution context  

This escalation was achieved without exploitation, indicating misconfiguration of privileged access.

---

### 3.3 Credential Dumping Activity

Following privilege escalation, the attacker accessed LSASS memory to extract credentials.

Observed techniques:
- LSASS memory access via Task Manager (online interaction)
- LSASS dump creation for offline credential extraction
- Potential use of credential extraction tools (e.g., Mimikatz)

---

### 3.4 Timeline

| Time | Event |
|------|------|
| 24/03/2026 20:45:27 | Login using helpdesk_admin on WS01 |
| 24/03/2026 20:45:36 | Process created with LSASS interaction |
| + Subsequent | Credential dumping activity performed |

---

## 4. Evidence and IOCs
The screenshots are provided in `lab-environment/credential-dumps-and-lateral-move/credential-dump-pivot.md`

---

## 5. Impact Assessment

- Systems affected: 1 (WS01)  
- Accounts exposed:
  - helpdesk_admin  
  - Potential additional cached credentials  

- Risk Level: High  
  - Credential exposure enables lateral movement  
  - Potential escalation to domain-level compromise  

---

## 6. Containment Actions

- Isolated WS01 from network  
- Reset credentials for helpdesk_admin and associated accounts  
- Reviewed privileged group memberships  
- Initiated credential hygiene measures across environment  

---

## 7. Root Cause

- Persistent local administrator privileges assigned to helpdesk_admin  
- Lack of monitoring for LSASS access activity  
- Insufficient restriction on credential access mechanisms  

---

## 8. MITRE ATT&CK Mapping

| Technique | ID |
|----------|----|
| Credential Dumping | T1003 |
| Valid Accounts | T1078 |
| Privilege Escalation | TA0004 |

---

## 9. Conclusion

The attacker successfully exploited administrative privilege exposure on WS01 to perform credential dumping via LSASS memory access. This activity significantly increases the risk of lateral movement and broader compromise. The incident highlights the importance of controlling privileged access and monitoring credential access behavior at the endpoint level.
