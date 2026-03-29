# Threat Intelligence Report  
## Lateral Movement Analysis: NotPetya Cyberattack (2017)


## 1. Executive Summary

The NotPetya cyberattack, observed in June 2017, leveraged a combination of credential theft and SMB-based lateral movement techniques to propagate rapidly across enterprise networks. Following initial access via a compromised software update mechanism, the malware utilized both valid account abuse and remote execution utilities to move laterally, enabling large-scale disruption within a short timeframe.

The campaign demonstrated how trusted administrative tools and stolen credentials can be combined to bypass traditional perimeter defenses and achieve widespread internal compromise.

---

## 2. Threat Overview

| Category | Details |
|--------|--------|
| Type | Destructive malware (wiper) |
| Initial Vector | Supply chain compromise |
| Primary Capability | Automated lateral propagation |
| Impact | Enterprise-wide disruption |

---

## 3. Technical Analysis

### 3.1 Initial Access

The infection originated through a compromised update mechanism of a widely used accounting software. Malicious code was executed within trusted environments, enabling attackers to establish an initial foothold without triggering perimeter defenses.

---

### 3.2 Credential Access

Post-compromise, the malware accessed the Local Security Authority Subsystem Service (LSASS) to extract credentials from memory. This included:

- NTLM hashes  
- Plaintext credentials (where available)

These credentials enabled authenticated access to additional systems within the network.

---

### 3.3 Lateral Movement

NotPetya employed multiple concurrent lateral movement techniques:

#### Valid Account Abuse
- Reused harvested credentials to authenticate to remote systems via SMB  
- Leveraged implicit trust relationships between hosts  

#### Remote Execution
- Executed commands on remote systems using:
  - PsExec  
  - Windows Management Instrumentation (WMIC)  

#### Exploit-Based Propagation
- Leveraged SMB vulnerability (EternalBlue) to compromise unpatched systems without requiring valid credentials  

---

### 3.4 Propagation Behavior

The combination of credential reuse and exploit-based techniques enabled rapid, automated propagation. Infected hosts initiated outbound connections to multiple internal systems, resulting in widespread compromise within a short period.

---

## 4. Observed Indicators

### 4.1 Authentication Activity
- Repeated network logons (Logon Type 3) across multiple hosts  
- Single account authenticating to numerous systems within a short timeframe  

### 4.2 Endpoint Activity
- Execution of administrative utilities:
  - psexec.exe  
  - wmic.exe  

### 4.3 Network Activity
- Increased SMB traffic (TCP port 445)  
- Elevated east-west traffic between internal systems  

### 4.4 Behavioral Indicators
- Rapid expansion from a single infected host  
- Simultaneous compromise of multiple endpoints  
- Absence of user-driven interaction post-infection  

---

## 5. Detection Opportunities

### Identity Monitoring
- Detection of abnormal authentication patterns  
- High-frequency logins across multiple hosts  

### Endpoint Monitoring
- Execution of remote administration tools in atypical contexts  
- Unauthorized access to LSASS memory  

### Network Monitoring
- Spikes in SMB traffic  
- Unusual lateral communication patterns between hosts  

### Behavioral Detection
- One-to-many host communication originating from a single system  
- Rapid increase in compromised endpoints  

---

## 6. Mitigation Strategies

- Enforce multi-factor authentication for privileged accounts  
- Restrict lateral movement through network segmentation  
- Monitor and limit use of administrative tools  
- Apply security patches to address SMB vulnerabilities  
- Implement endpoint detection for credential access behavior  

---

## 7. MITRE ATT&CK Mapping

| Technique | ID |
|----------|----|
| Supply Chain Compromise | T1195 |
| Credential Dumping | T1003 |
| Remote Services | T1021 |
| Valid Accounts | T1078 |
| Windows Management Instrumentation | T1047 |
| Exploitation of Remote Services | T1210 |

Reference: MITRE ATT&CK Framework

---

## 8. Assessment

The NotPetya attack highlights the critical role of lateral movement in scaling impact. Once initial access is achieved, the ability to reuse credentials and leverage built-in administrative tools significantly reduces the need for additional exploitation, allowing attackers to expand rapidly within the network.

Organizations with limited visibility into internal authentication and host-to-host communication remain at elevated risk of similar propagation techniques.

---

## 9. Conclusion

NotPetya demonstrates that lateral movement is the primary force multiplier in enterprise compromise. The use of valid credentials and trusted tools enables attackers to operate within expected system behavior, complicating detection efforts and accelerating impact.

Effective defense requires a combination of identity monitoring, endpoint visibility, and network-level detection to identify and contain lateral movement activity before it escalates.

---

## 10. References

- Cisco Talos — "Nyetya: A MBR wiping ransomware that uses EternalBlue"  
- Microsoft — "Petya ransomware outbreak: Here’s what you need to know"  
- Mandiant — "NotPetya Malware Technical Analysis"  
- US-CERT — "Alert (TA17-181A): Petya Ransomware"  
- ESET — "Diskcoder.C Technical Analysis"  
- MITRE ATT&CK Framework  
