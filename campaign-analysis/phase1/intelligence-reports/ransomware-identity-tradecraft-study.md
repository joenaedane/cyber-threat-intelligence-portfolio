# Incident Report - Identity-Driven Intrusion Reconstruction Based on Ryuk Ransomware Tradecraft

## Author's Notes
This case study is reconstructing a simulated intrusion inspired by the attack patterns associated with Ryuk Ransom Attacks. This case study aims to provide current threat actors' possible attack vectors.

Ryuk campaigns typically begin with phishing emails carrying the Emotet or TrickBot trojans. Once inside a network, attackers move laterally, harvest credentials, and disable backups before deploying Ryuk. The ransomware encrypts files using robust algorithms (AES and RSA), appending “.ryk” or similar extensions and dropping ransom notes demanding cryptocurrency payments.

This case study is reconstructing a simulated intrusion inspired by the attack patterns associated with Ryuk Ransom Attacks. It will be diving into the possibility of Ryuk leverage credential access techniques such as Kerberoasting, lateral movement using privileged service accounts, and backup infrastructure compromise to prepare environments for ransomware deployment.


## Incident Overview
Incident Type: Identity-based lateral movement / potential ransomware staging
Severity: High
Primary Affected Assets: Backup Server, File Server, SQL Infrastructure
Suspected Technique: Kerberoasting-style credential access
Analysis Type: Simulated reconstruction based on known ransomware tradecraft


## Executive Summary
Analysis indicates abnormal Kerberos service ticket activity associated with a privileged backup service account.

Subsequent authentication events suggest lateral movement across enterprise infrastructure systems and modification of backup operations.

Observed behaviours are consistent with identity-driven intrusion patterns commonly preceding ransomware deployment.

## Observed Sequence of Malicious Activity

Security telemetry indicates a probable identity-driven intrusion involving Kerberos service ticket abuse and subsequent lateral movement across enterprise infrastructure.

Initial anomalous activity was identified through an elevated volume of Kerberos Service Ticket Request events (Event ID 4769) associated with the service account svc_backup. Ticket requests were observed targeting multiple Service Principal Names (SPNs), including database, file sharing, and application hosting services.

Following this anomaly, successful authentication events (Event ID 4624 — Logon Type 3 / Network) were observed originating from the same service account toward multiple infrastructure servers. The authentication source was identified as a standard user workstation rather than a designated backup management host, suggesting possible credential misuse.

Subsequent host telemetry and administrative logs indicated modification of scheduled backup tasks and execution of scripts associated with backup deletion or recovery inhibition. These activities are consistent with attacker attempts to weaken organizational resilience mechanisms prior to impact operations such as ransomware deployment.

Additional risk indicators include potential exposure of privileged authentication material due to administrative logons recorded on the affected backup server during the intrusion window. This increases the likelihood of credential harvesting or Kerberos ticket abuse for further privilege escalation.

Collectively, the sequence of observed indicators supports an assessment of probable Kerberoasting-enabled credential compromise, followed by lateral movement using valid service account authentication, and pre-impact manipulation of backup infrastructure.

## Observed Evidence Patterns
 * Abnormal Kerberos Ticket Requests
   - Elevated volume of Kerberos Service Ticket Request events (Event ID 4769)
   - Requests targeting multiple Service Principal Names including database and file services
   - Service account authentication originating from non-standard workstation
   - Implications: Attackers are enumerating through services and getting Kerberos Service Ticket to retrieve Service Account Password for a Silver Ticket

* Privileged Infrastructure Access
    - Successful network logons recorded to backup and application servers
    - Access patterns deviating from established behavioural baseline
    - Implications: Attackers had successfully lateral moved to target and preparation for execution is immittent. 

* Backup Configuration Manipulation
    - Creation or modification of scheduled tasks related to backup execution
    - Evidence of backup deletion or recovery inhibition activity
    - Implications: Attackers have impacted the data's integrity and availability. 

## Evidence Highlights
# Kerberos Service Ticket Requests (Event ID 4769)
```
LogName: Security
EventID: 4769
Computer: DC01.corp.local
Account Name: svc_backup
Service Name: MSSQLSvc/finance-db.corp.local:1433
Ticket Encryption Type: 0x17
Client Address: 10.10.22.54
Failure Code: 0x0
Ticket Options: 0x40810000
```
```
LogName: Security
EventID: 4769
Computer: DC01.corp.local
Account Name: svc_backup
Service Name: CIFS/fileserver01.corp.local
Ticket Encryption Type: 0x17
Client Address: 10.10.22.54
Failure Code: 0x0
Ticket Options: 0x40810000
```
```
LogName: Security
EventID: 4769
Computer: DC01.corp.local
Account Name: svc_backup
Service Name: HOST/appserver02.corp.local
Ticket Encryption Type: 0x17
Client Address: 10.10.22.54
Failure Code: 0x0
Ticket Options: 0x40810000
```
* Multiple unique SPN requests from the same source workstation
* Service account normally does not access these services


# Successful Network Logons (Event ID 4624)
```
LogName: Security
EventID: 4624
Computer: backup01.corp.local
Account Name: svc_backup
Logon Type: 3
Source Network Address: 10.10.22.54
Authentication Package: Kerberos
Process Name: -
```
```
LogName: Security
EventID: 4624
Computer: fileserver01.corp.local
Account Name: svc_backup
Logon Type: 3
Source Network Address: 10.10.22.54
Authentication Package: Kerberos
```
* Service account authenticating from user workstation subnet
* Indicates credential misuse or ticket abuse and suggests post-compromise lateral movement


# Scheduled Task Creation (Event ID 4698)
```
LogName: Security
EventID: 4698
Computer: backup01.corp.local
Task Name: WindowsBackupMaintenance
Author: svc_backup
Action: powershell.exe -ExecutionPolicy Bypass -File cleanup-backup.ps1
```
* New scheduled task created by service account
* PowerShell execution linked to backup scripts
* Potential persistence or defense evasion preparation

# Backup Deletion Activity
```
Source: BackupExec
EventID: 34113
Message: Backup set deletion initiated by user svc_backup
Target Path: \\backuprepo01\vault\weekly\
Status: Success
```
* Backup deletion outside normal maintenance window
* Strong ransomware precursor indicator

## Potential Impact Assessment
 * Increased risk of enterprise data unavailability
 * Exposure of sensitive operational data
 * Privilege escalation pathways toward domain-level compromise
 * Disruption of business continuity mechanisms
 * Backup infrastructure compromise significantly elevates overall incident severity.
 
## MITRE ATT&CK Technique Mapping
 * Discovery	
    - Account Discovery (T1087)
    - Phishing (T1566)
 * Credential Access	
    - Kerberoasting (T1558.003)
    - OS Credential Dumping (T1003)
 * Lateral Movement	
    -  Remote Services (T1021)
 * Persistence	
    - Scheduled Task/Job (T1053)
 * ImpacT
    - Inhibit System Recovery (T1490)

## Containment Actions Recommended
 * Immediate
    - Isolate affected servers from network
    - Reset and rotate compromised service account credentials
    - Invalidate active Kerberos tickets where feasible
 * Short Term
    - Audit delegation configurations across privileged systems
    - Review service account privilege scope
    - Validate integrity of backup repositories
 * Long Term
    - Implement managed service accounts or credential vaulting
    - Deploy behavioural detection analytics for Kerberos telemetry
    - Harden identity monitoring across Tier-0 infrastructure

## Related Tradecraft & Real-World Campaign References

Identity-centric intrusion patterns involving Kerberos abuse, credential dumping, and backup infrastructure targeting have been documented across multiple ransomware and advanced intrusion campaigns.

### NotPetya cyberattack

Observed TTPs:

Rapid lateral movement using credential reuse
Abuse of legitimate administration tools
Enterprise-wide impact through identity trust exploitation

This campaign demonstrates how identity compromise can enable large-scale operational disruption without reliance on traditional exploitation.

### SolarWinds supply chain attack

Observed TTPs:

Long dwell time with stealthy identity abuse
Privileged service account compromise
Use of legitimate authentication mechanisms to avoid detection

This reinforces the importance of behavioural monitoring of Kerberos authentication activity.