# Incident Report: Ransomware-Style Impact via File Modification and Recovery Inhibition on Domain Controller (DC01)

---

## Source of Analysis

This incident assessment is derived from behavioural observations recorded during a simulated attack path analysis exercise conducted within a controlled Active Directory lab environment.

Technical Simulation Reference:
`lab-environment/impact/ransomware-simulation.md`

---

## 1. Incident Overview

High-impact system activity was observed on the Domain Controller (DC01), including mass file modification, recovery inhibition attempts, and deployment of ransom artifacts. Investigation confirmed that the attacker executed ransomware-style operations using native system tools to simulate file encryption and disrupt system functionality.

The attacker leveraged an existing privileged service account (`svc_backup`) to perform bulk file modification, append new file extensions, and overwrite original data. Additional actions included deletion attempts of shadow copies and disruption of system services.

The incident is classified as Critical severity due to demonstrated capability to cause operational disruption and irreversible data loss on a domain-critical system.

---

## 2. Initial Detection

The incident was identified through abnormal file system activity and execution of administrative utilities on DC01.

Key indicators:

* Rapid modification and renaming of multiple files
* Creation of ransom note (`README_RECOVER.txt`)
* Execution of `vssadmin` to delete shadow copies
* Service disruption activity (e.g., spooler stopped)

Relevant telemetry sources:

* Windows Security Logs (Event ID 4688 — process execution, if enabled)
* File system monitoring
* Host-based observation

---

## 3. Investigation Findings

### 3.1 Mass File Modification (Encryption Simulation)

* Target Directory: `C:\impact_test`
* Technique: Bulk overwrite and file renaming

Observed behavior:

* Multiple files modified within a short time window
* File contents replaced with placeholder data ("ENCRYPTED DATA")
* File extensions changed to `.locked`

This behaviour is consistent with ransomware encryption activity, where original data becomes inaccessible.

---

### 3.2 Ransom Note Deployment

* File: `README_RECOVER.txt`

Observed behavior:

* Ransom note created within affected directory
* Message indicates data encryption and payment demand

This confirms attacker intent and provides victim instructions, a standard ransomware characteristic.

---

### 3.3 Recovery Inhibition

* Command executed: `vssadmin delete shadows /all /quiet`

Observed behavior:

* Attempted deletion of shadow copies to prevent recovery
* No shadow copies present on system

This reflects standard ransomware behaviour, even when recovery mechanisms are not available.

---

### 3.4 Service Disruption

* Service affected: Print Spooler

Observed behavior:

* Service manually stopped
* Potential disruption to system functionality

This demonstrates attacker capability to impact business operations beyond data encryption.

---

### 3.5 Privileged Execution Context

* Account Used: `corp\svc_backup`

Observed behavior:

* All actions executed under Domain Administrator privileges
* No restrictions on system-level operations

This highlights the risk of privileged account compromise in enabling high-impact attacks.

---

### 3.6 Timeline

| Time     | Event                                        |
| -------- | -------------------------------------------- |
| 4/13/2026 19:30:00  | Test files created in target directory       |
| 4/13/2026 19:31:00 | Bulk file modification and renaming observed |
| 4/13/2026 19:32:00 | Ransom note deployed                         |
| 4/13/2026 19:34:00 | Shadow copy deletion attempted               |
| 4/13/2026 19:35:00 | Service disruption executed                  |

The rapid sequence of events indicates automated or scripted execution rather than manual interaction.

---

## 4. Evidence and IOCs

* Directory:

  * `C:\impact_test`

* Files:

  * `*.locked` (modified files)
  * `README_RECOVER.txt`

* Commands:

  * PowerShell file modification script
  * `vssadmin delete shadows /all /quiet`
  * `net stop spooler`

* Indicators:

  * Sudden spike in file changes
  * Ransom note presence
  * Recovery mechanism tampering

---

## 5. Impact Assessment

* Systems affected:

  * DC01 (Domain Controller)

* Accounts involved:

  * svc_backup

* Risk Level: Critical

  * Data rendered inaccessible
  * Recovery mechanisms disabled
  * Operational disruption introduced

---

## 6. Containment Actions

* Terminated malicious PowerShell execution
* Restored affected services
* Re-enabled recovery mechanisms (if applicable)
* Isolated affected systems
* Reviewed privileged account usage

---

## 7. Root Cause

* Over-privileged service account with unrestricted access
* Lack of monitoring for bulk file operations
* Absence of detection for recovery inhibition commands
* Limited visibility into PowerShell activity

---

## 8. MITRE ATT&CK Mapping

| Technique                         | ID    |
| --------------------------------- | ----- |
| Data Encrypted for Impact         | T1486 |
| Inhibit System Recovery           | T1490 |
| Command and Scripting Interpreter | T1059 |
| Valid Accounts                    | T1078 |

---

## 9. Conclusion

The attacker successfully simulated ransomware-style impact using native system tools to modify files, deploy ransom artifacts, and disrupt recovery mechanisms.

This demonstrates how attackers can achieve significant operational disruption without deploying traditional malware, relying instead on legitimate tools and privileged access.

The incident highlights the importance of monitoring file system activity, privileged account usage, and recovery-related commands to detect and mitigate ransomware attacks.

---
