# Kerberoasting Activity

## Technique Overview
Kerberoasting is a credential access technique in which an adversary leverages valid domain user access to request Kerberos service tickets associated with Service Principal Names (SPNs).

These service tickets are encrypted using the password-derived key of the target service account. An attacker can extract the ticket material and perform offline password cracking to recover the underlying service account credential.

Successful compromise of service accounts may enable privilege escalation, lateral movement across infrastructure systems, and persistence through forged Kerberos tickets. Because ticket requests are legitimate Kerberos operations, Kerberoasting activity can blend with normal authentication traffic and may evade detection without behavioural analysis.

## Detection Objective
Detect potential Kerberoasting activity by identifying abnormal Kerberos service ticket request patterns that deviate from established behavioural baselines.

The detection aims to identify:
* Service account credential harvesting
* Identity reconnaissance activity
* Early-stage lateral movement preparation

## Detection Hypothesis

A user or service account requesting multiple unique Service Principal Names (SPNs) within a short time window, especially from a non-standard host, may indicate Kerberoasting or Kerberos reconnaissance activity.

## Primary Data Source
Windows Security Event Logs
Domain Controller telemetry

Key Event:
```
Event ID 4769 — Kerberos Service Ticket Request
```

## Correlated Alert (Behaviour based)

### Stage 1 Correlation
* Same account requests ≥ X unique SPNs
* Within Y minutes
* From same source IP / workstation
* AND account baseline normally accesses fewer services

### Stage 2 Correlation
* Detect Event ID 4624 after Kerberos ticket spike

### Stage 3 Correlation
* Scheduled task creation (Event ID 4698)
* Service installation (Event ID 7045)
* Backup deletion logs


## Enrichment Conditions (Increase Confidence)

Alert severity increases if:

* Account is service account
* Activity occurs outside business hours
* Source host is user workstation subnet
* Followed by Event ID 4624 lateral logons
* Followed by Event ID 4672 privileged logon

## False Positive Considerations

Legitimate causes may include:
- Application deployment
- Backup job reconfiguration
- Patch cycles
- Monitoring system scanning
- Bulk service startup

## Severity Scoring Model
* Requests to more than 5 SPNs +2
* Service Account Involved +2
* Non-standard workstation source +2
* After-hours/maintenance activity +1
* Follow-on lateral movement +3

## Response Playbook Trigger

If alert confirmed:

- Validate account baseline
- Identify affected hosts
- Check for credential dumping telemetry
- Force password reset
- Purge Kerberos tickets
- Initiate lateral movement hunt