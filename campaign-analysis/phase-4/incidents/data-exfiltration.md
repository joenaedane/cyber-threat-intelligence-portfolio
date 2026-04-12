# Incident Report: Data Exfiltration via Scheduled Low-and-Slow Transfer on Domain Controller (DC01)

## Source of Analysis

This incident assessment is derived from behavioural observations
recorded during a simulated attack path analysis exercise conducted
within a controlled Active Directory lab environment.

Technical Simulation Reference:\
`lab-environment/exfiltration/scheduled-exfiltration.md`

## 1. Incident Overview

Suspicious outbound network activity and file manipulation were observed
on the Domain Controller (DC01), indicating data exfiltration.
Investigation confirmed that the attacker staged sensitive data,
compressed it into an archive, and leveraged a scheduled task to
periodically transfer the data to an external system.

The attacker operated using a privileged service account (`svc_backup`)
and native system utilities, enabling stealthy data exfiltration while
minimizing detection.

The incident is classified as Critical severity due to successful data
extraction from a domain-critical asset using low-frequency, automated
transfer mechanisms.

## 2. Initial Detection

The incident was identified through abnormal file activity and recurring
outbound network connections from DC01.

Key indicators: - Creation of archive file (`C:\temp\data.zip`)\
- Movement of sensitive data into staging directory\
- Scheduled task executing PowerShell-based network transfer\
- Repeated outbound connections to non-corporate IP address

Relevant telemetry sources: - Windows Security Logs (Event IDs: 4688,
4698, 4624, 4672 --- if enabled)\
- File system activity (host-based observation)\
- Network connection behaviour

## 3. Investigation Findings

### 3.1 Data Staging and Preparation

-   Target Host: DC01\
-   File: `C:\temp\data.zip`

Observed behavior: - Sensitive data created and moved into staging
directory (`C:\temp\staging`)\
- Data compressed into archive using PowerShell (`Compress-Archive`)

### 3.2 Scheduled Exfiltration Mechanism

-   Mechanism: Scheduled Task (`SystemTelemetryService`)\
-   Execution Method: PowerShell TCP-based transfer

Observed behavior: - Scheduled task created to execute at regular
intervals\
- PowerShell script reads archive and transmits data externally

### 3.3 Network Behaviour

-   Source Host: DC01\
-   Destination: Attacker-controlled IP\
-   Port: 5555

Observed behavior: - Repeated outbound connections to the same external
IP\
- Consistent interval-based communication (\~5 minutes)

### 3.4 Privileged Execution Context

-   Account Used: `corp\svc_backup`

### 3.5 Logging Limitations

-   Limited process and task logging\
-   Weak native network visibility

### 3.6 Timeline

  Time       Event
  ---------- ------------------------
  14:43:15   Sensitive data staged
  14:44:21   Archive created
  14:47:32   Scheduled task created
  14:52:32   First exfiltration
  14:57:32   Repeated connection
  15:02:32   Continued transfer

## 4. Evidence and IOCs

-   File: `C:\temp\data.zip`
-   Task: `SystemTelemetryService`
-   External IP (attacker)
-   Port 5555

## 5. Impact Assessment

Critical --- Data successfully exfiltrated from Domain Controller.

## 6. Containment Actions

-   Removed scheduled task\
-   Blocked outbound connection\
-   Cleaned staged data

## 7. Root Cause

-   Lack of monitoring\
-   Over-privileged account\
-   No outbound restrictions

## 8. MITRE ATT&CK Mapping

-   T1041\
-   T1053\
-   T1059\
-   T1560\
-   T1078

## 9. Conclusion

The attacker successfully performed stealthy exfiltration using
scheduled low-frequency transfer, highlighting detection gaps in
identity, task execution, and network monitoring.
