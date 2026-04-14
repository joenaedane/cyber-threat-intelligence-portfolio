# Full Campaign Intelligence Assessment

## Enterprise Intrusion Lifecycle: Identity Compromise to Ransomware Impact

---

## Campaign Context

This intelligence assessment reconstructs a simulated enterprise intrusion spanning credential access, lateral movement, persistence, command-and-control (C2), data exfiltration, and ransomware-style impact.

The campaign demonstrates how attackers leverage identity-based techniques and native system tools to progress from initial compromise to full operational control and disruption.

Rather than relying on software exploits, the intrusion is driven by **misconfigured privileges, credential reuse, and trusted system functionality**.

---

## Campaign Overview

The attacker progresses through the following stages:

```plaintext
Initial access → Credential access → Lateral movement → Domain control
→ Persistence → Command-and-control → Data exfiltration → Impact
```

This sequence reflects a structured, multi-phase intrusion aligned with modern adversary tradecraft.

---

## Intrusion Activity Reconstruction

### 1. Credential Access and Lateral Movement (Phase 2)

The attacker gains access to LSASS memory on WS01 and extracts reusable authentication material.

Observed behaviour:

* Credential dumping via memory access
* Extraction of NTLM hashes
* Authentication to DC01 via Pass-the-Hash

This enables lateral movement without requiring plaintext credentials.

---

### 2. Domain Control via Identity Abuse (Phase 3)

The attacker establishes control over Active Directory by manipulating identity structures.

Observed behaviour:

* Creation of service accounts (`svc_backup`, `svc_ops`)
* Addition to Domain Admins group
* Privileged authentication across systems

This represents a shift from access to **identity-layer dominance**.

---

### 3. Persistence and Command-and-Control

The attacker ensures continued access through:

* Scheduled task execution
* PowerShell-based reverse communication
* Outbound beaconing to attacker-controlled infrastructure

This enables continuous remote control without reliance on initial access vectors.

---

### 4. Data Exfiltration (Phase 4 — Lab 1)

Sensitive data is staged, compressed, and transferred externally.

Observed behaviour:

* Data staging in temporary directories
* Archive creation (`data.zip`)
* Scheduled low-frequency outbound transfer

This demonstrates **low-and-slow exfiltration**, minimizing detection.

---

### 5. Ransomware-Style Impact (Phase 4 — Lab 2)

The attacker executes high-impact operations using native tools.

Observed behaviour:

* Bulk file modification and renaming (`.locked`)
* Ransom note deployment
* Shadow copy deletion attempt (`vssadmin`)
* Service disruption

This marks the transition from stealth to **overt operational impact**.

---

## Intelligence Assessment

The campaign demonstrates a complete intrusion lifecycle driven by identity compromise and native tooling.

Key characteristics:

* Reliance on valid credentials and administrative access
* Minimal use of external malware
* Use of scheduled tasks and PowerShell for persistence and execution
* Combination of stealth (early phases) and noise (impact phase)

This behaviour aligns with modern ransomware operations such as LockBit and similar RaaS groups.

---

## Detection Opportunities

### Identity Monitoring

* Abnormal privileged account creation
* Group membership changes

---

### Endpoint Monitoring

* PowerShell execution
* Bulk file modification
* Execution of `vssadmin`

---

### Network Monitoring

* Repeated outbound connections
* Exfiltration patterns

---

### Behavioural Correlation

* Sequence of credential access → lateral movement → persistence
* Transition from low activity to high-impact behaviour

---

## Defensive Blind Spots Identified

* Lack of monitoring for credential dumping
* Insufficient visibility into scheduled task abuse
* Limited outbound network monitoring
* Weak detection of file staging and compression
* Over-reliance on individual alerts rather than correlation

---

## MITRE ATT&CK Mapping

* Credential Dumping (T1003)
* Pass-the-Hash (T1550.002)
* Account Manipulation (T1098)
* Scheduled Task/Job (T1053)
* Command and Scripting Interpreter (T1059)
* Exfiltration Over C2 Channel (T1041)
* Data Encrypted for Impact (T1486)
* Inhibit System Recovery (T1490)

---

## Campaign Conclusion

The attacker successfully transitioned from initial compromise to full domain control, data exfiltration, and operational disruption.

The intrusion highlights that:

* Identity compromise is the central enabler of enterprise attacks
* Native tools can be leveraged to evade detection
* Detection requires correlation across multiple phases

Without effective monitoring and response, such attacks can progress undetected until the impact stage, where remediation becomes significantly more difficult.

---
