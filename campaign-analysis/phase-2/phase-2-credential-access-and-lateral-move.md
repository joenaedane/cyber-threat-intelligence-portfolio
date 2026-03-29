# Credential Access & Lateral Movement

---

## Campaign Context

This intelligence assessment reconstructs the second phase of a simulated enterprise intrusion focused on credential harvesting and lateral movement using compromised authentication material.

Following identity reconnaissance and escalation preparation in Phase-1, the attacker transitions into active exploitation of privilege exposure to obtain reusable credentials and expand access across the environment.

This phase demonstrates how identity compromise enables controlled progression toward domain-level compromise without reliance on software exploitation.

---

## Phase Transition Overview

The attacker enters this phase with:

- Established foothold on workstation (WS01)  
- Knowledge of privilege exposure (helpdesk_admin)  
- Identified escalation pathway from prior reconnaissance  

The objective shifts from reconnaissance to **credential acquisition and operational expansion**.

---

## Phase Objectives Observed

Adversary activity during this phase indicates intent to:

- Extract authentication material from memory  
- Leverage administrative privileges for credential access  
- Reuse stolen credentials for cross-host authentication  
- Establish control over critical infrastructure (Domain Controller)  
- Transition from local compromise to domain-level access  

These objectives reflect execution of previously identified escalation pathways.

---

## Intrusion Activity Timeline Reconstruction

### 1. Privilege Escalation via Administrative Exposure

The attacker leverages persistent local administrative privileges assigned to helpdesk_admin on WS01.

This enables elevation to a high-integrity execution context without exploitation of vulnerabilities.

Such privilege exposure represents a common enterprise misconfiguration that facilitates attacker progression.

---

### 2. Credential Access via LSASS Memory Interaction

With administrative privileges, the attacker accesses LSASS memory to extract authentication material.

Observed behavior includes:

- Memory access targeting LSASS  
- Extraction of NTLM hashes  
- Potential offline parsing of credential data  

This activity enables acquisition of reusable authentication material without triggering password-based detection mechanisms.

---

### 3. Credential Reuse via Pass-the-Hash

The attacker utilizes extracted NTLM hashes to authenticate to remote systems without knowledge of plaintext credentials.

This technique allows authentication to proceed through legitimate protocol mechanisms, reducing the likelihood of detection.

Authentication activity originates from WS01 and targets domain infrastructure.

---

### 4. Lateral Movement to Domain Controller

Using Pass-the-Hash, the attacker authenticates to DC01 over SMB and executes remote commands via service creation.

This results in:

- Network logon (Logon Type 3) to Domain Controller  
- Assignment of privileged access (Administrator context)  
- Remote execution under SYSTEM privileges  

This marks a critical escalation point where control of domain infrastructure is achieved.

---

## Intelligence Assessment

This phase demonstrates a controlled transition from credential access to lateral movement using identity-based techniques.

Rather than exploiting vulnerabilities, the attacker leverages:

- Misconfigured administrative privileges  
- Reusable authentication material  
- Trusted communication protocols (SMB, NTLM)  

The linkage between LSASS credential extraction on WS01 and subsequent authentication to DC01 highlights a deterministic escalation path enabled entirely through identity compromise.

This behavior is consistent with modern intrusion campaigns where attackers prioritize stealth, reliability, and reuse of legitimate access mechanisms.

---

## Defensive Blind Spots Identified

- Limited monitoring of LSASS memory access  
- Lack of detection for credential dumping activity  
- Continued reliance on NTLM authentication  
- Insufficient visibility into internal authentication patterns  
- Weak control over administrative privilege assignment  
- Lack of correlation between credential access and lateral movement events  

These gaps allow attackers to transition between phases without triggering high-confidence alerts.

---

## MITRE ATT&CK Mapping (Phase-2)

- **Credential Dumping (T1003)**  
- **Pass-the-Hash (T1550.002)**  
- **Remote Services (T1021)**  
- **Valid Accounts (T1078)**  
- **Privilege Escalation via Valid Accounts (TA0004)**  

---

## Phase Conclusion

The attacker has successfully transitioned from credential access to lateral movement, achieving privileged execution on the Domain Controller.

This represents a critical inflection point in the intrusion where domain-level compromise becomes achievable.

If defensive intervention is not performed, the next likely attacker objective involves persistence establishment, broader lateral expansion, or domain dominance activities.