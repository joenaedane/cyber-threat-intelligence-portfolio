# Incident Report: Lateral Movement via Pass-the-Hash to Domain Controller (DC01)

---


## Source of Analysis
This incident assessment is derived from behavioural observations recorded during a simulated attack path analysis exercise conducted within a controlled Active Directory lab environment.

Technical Simulation Reference:
`lab-environment/credential-dumps-and-lateral-move/lateral-move.md`


## 1. Incident Overview

Suspicious authentication activity was detected involving a workstation (WS01) initiating privileged access to the Domain Controller (DC01). Investigation confirmed that the attacker leveraged stolen NTLM authentication material to perform Pass-the-Hash (PtH), enabling unauthorized lateral movement and remote command execution on DC01. The incident is classified as Critical severity due to the successful compromise of domain-level infrastructure.

---

## 2. Initial Detection

The incident was triggered by abnormal authentication behavior observed on DC01.

Key indicators:
- Network logon (Logon Type 3) from workstation WS01
- Use of privileged account (Administrator)
- Authentication without corresponding interactive login behavior

Relevant telemetry sources:
- Windows Security Logs (Event IDs: 4624, 4672)
- Endpoint and service execution logs

---

## 3. Investigation Findings

### 3.1 Credential Source

- Origin Host: WS01  
- Credential Type: NTLM hash  
- Source: Extracted from LSASS memory (prior incident)

The attacker obtained authentication material from a compromised workstation and reused it without needing plaintext credentials.

---

### 3.2 Lateral Movement Activity

- Source Host: WS01  
- Target Host: DC01  
- Account Used: Administrator  

Observed technique:
- Pass-the-Hash authentication over SMB (port 445)

The attacker authenticated to DC01 using NTLM hash material and established a remote session.

---

### 3.3 Remote Execution

Following successful authentication, the attacker executed commands on DC01 using remote service creation.

Observed behavior:
- Service binary written to ADMIN$ share  
- Remote service created and executed  
- Commands executed under SYSTEM-level context  

---

### 3.4 Timeline

| Time | Event |
|------|------|
| 25/03/2026 10:34:42 | NTLM-based logon from WS01 to DC01 |
| 25/03/2026 10:34:42 | Privileged logon assigned (Event ID 4672) |
| 25/03/2026 10:37:21 | Remote service execution initiated |

---

## 4. Evidence and IOCs
The screenshots are provided in `lab-environment/credential-dumps-and-lateral-move/lateral-move.md`


---

## 5. Impact Assessment

- Systems affected:
  - WS01 (source)
  - DC01 (compromised)

- Accounts involved:
  - Administrator

- Risk Level: Critical  
  - Domain Controller access achieved  
  - Potential for full domain compromise  
  - Ability to deploy persistence and expand control  

---

## 6. Containment Actions

- Disabled compromised accounts  
- Isolated WS01 from network  
- Investigated DC01 for persistence mechanisms  
- Reset privileged credentials across domain  
- Restricted SMB-based administrative access  

---

## 7. Root Cause

- Exposure of NTLM authentication material via LSASS dumping  
- Lack of credential protection controls  
- Over-reliance on NTLM authentication  
- Insufficient monitoring of internal authentication behavior  

---

## 8. MITRE ATT&CK Mapping

| Technique | ID |
|----------|----|
| Pass-the-Hash | T1550.002 |
| Remote Services (SMB/PsExec) | T1021 |
| Valid Accounts | T1078 |
| Credential Dumping (Precursor) | T1003 |

---

## 9. Conclusion

The attacker successfully leveraged stolen NTLM authentication material to perform lateral movement from WS01 to DC01 using Pass-the-Hash. This enabled remote command execution with elevated privileges on the Domain Controller, representing a critical escalation point in the attack chain.

The NTLM hash leveraged in this incident was obtained from prior credential dumping activity on WS01 (Incident-01), demonstrating direct linkage between credential access and lateral movement phases.

The incident demonstrates how credential compromise directly enables network traversal and highlights the risks associated with NTLM-based authentication and insufficient internal monitoring.
