# Incident Report: Command-and-Control Establishment via PowerShell Beacon on Domain Controller (DC01)

---

## Source of Analysis

This incident assessment is derived from behavioural observations recorded during a simulated attack path analysis exercise conducted within a controlled Active Directory lab environment.

Technical Simulation Reference:
`lab-environment/persistence-c2/c2-establishment.md`

---

## 1. Incident Overview

Suspicious outbound communication and recurring PowerShell execution were observed on the Domain Controller (DC01), indicating the establishment of a command-and-control (C2) channel. Investigation confirmed that the attacker deployed a PowerShell-based reverse beacon and configured scheduled execution to maintain continuous remote access.

The attacker leveraged a previously established privileged account (`svc_backup`) to execute commands and maintain control over the domain asset.

The incident is classified as Critical severity due to the transition from persistent access to **active attacker-controlled operations** within the domain environment.

---

## 2. Initial Detection

The incident was identified through abnormal outbound communication patterns and execution of PowerShell scripts on DC01.

Key indicators:

* Repeated outbound connections to a non-corporate IP address
* Execution of PowerShell script (`beacon.ps1`)
* Creation of scheduled task for recurring execution
* Privileged account (`svc_backup`) associated with activity

Relevant telemetry sources:

* Windows Security Logs (Event IDs: 4688, 4698, 4624, 4672)
* Network connection observations (host-based)

---

## 3. Investigation Findings

### 3.1 Command-and-Control Mechanism

* Target Host: DC01
* Account Used: corp\svc_backup
* Technique: PowerShell-based reverse TCP beacon

Observed behavior:

* PowerShell script (`beacon.ps1`) executed on DC01
* Outbound connection initiated from DC01 to attacker-controlled system
* Interactive command execution confirmed (e.g., `whoami → svc_backup`)

The attacker established a reverse communication channel, enabling remote command execution without requiring inbound connectivity.

---

### 3.2 Persistence of C2 Channel

* Mechanism: Scheduled Task

Observed behavior:

* Scheduled task (`WindowsUpdateService`) created (Event ID 4698 — if logging enabled)
* Task configured to execute PowerShell beacon at regular intervals
* Beacon re-establishes connection upon termination

This ensures continuous availability of the C2 channel, even after system restarts or session termination.

---

### 3.3 Privileged Execution Context

* Account Used: corp\svc_backup

Observed behavior:

* Successful privileged logon (Event ID 4624)
* Assignment of administrative privileges (Event ID 4672)
* Execution of C2 activity under Domain Admin context

This confirms that the attacker maintains full administrative control while operating through the C2 channel.

---

### 3.4 Network Behaviour

* Source Host: DC01
* Destination: Attacker-controlled IP
* Port: 4444

Observed behavior:

* Repeated outbound TCP connections
* Short-lived, periodic communication patterns
* No legitimate business justification for external communication

This pattern is consistent with **beaconing activity used in command-and-control frameworks**.

---

### 3.5 Logging Limitations

Observed gaps:

* Process creation logs (Event ID 4688) not consistently available
* Scheduled task creation logs (Event ID 4698) dependent on audit policy
* Limited native visibility into network connections

These gaps highlight reliance on enhanced logging (e.g., Sysmon) for full detection capability.

---

### 3.6 Timeline

| Time      | Event                                   |
| --------- | --------------------------------------- |
| 12:10:00  | PowerShell beacon executed on DC01      |
| 12:11:00  | Initial outbound connection established |
| 12:15:00  | Scheduled task created for persistence  |
| 12:20:00+ | Repeated beaconing activity observed    |

---

## 4. Evidence and IOCs

* Suspicious script: `C:\beacon.ps1`
* Scheduled Task: `WindowsUpdateService`
* External IP: Attacker-controlled host
* Port: 4444

---

## 5. Impact Assessment

* Systems affected:

  * DC01 (Domain Controller)

* Accounts involved:

  * svc_backup

* Risk Level: Critical

  * Continuous remote command execution capability
  * Persistent control channel established
  * Enables further attacker operations (lateral expansion, exfiltration, impact)

---

## 6. Containment Actions

* Terminated malicious PowerShell processes
* Removed scheduled task (`WindowsUpdateService`)
* Disabled or reset `svc_backup` account credentials
* Blocked outbound communication to attacker IP
* Enabled enhanced logging for process and network activity

---

## 7. Root Cause

* Lack of monitoring on PowerShell execution
* Absence of detection for scheduled task abuse
* Excessive privileges assigned to service account
* Insufficient network monitoring for outbound anomalies

---

## 8. MITRE ATT&CK Mapping

| Technique                         | ID    |
| --------------------------------- | ----- |
| Command and Control               | T1071 |
| Command and Scripting Interpreter | T1059 |
| Scheduled Task/Job                | T1053 |
| Valid Accounts                    | T1078 |

---

## 9. Conclusion

The attacker successfully transitioned from persistence to **active command-and-control operations** by deploying a PowerShell-based reverse beacon on the Domain Controller.

The use of outbound communication allowed the attacker to bypass traditional network restrictions, while scheduled execution ensured resilience of the control channel.

Operating under a privileged service account further reduced detection visibility and enabled unrestricted control over the environment.

This incident demonstrates how attackers evolve from initial access to sustained operational control, emphasizing the importance of monitoring script execution, scheduled task creation, and anomalous outbound network activity within domain infrastructure.
