# Threat Intelligence Report

## Persistence & Command-and-Control Analysis: SolarWinds Supply Chain Compromise (SUNBURST, 2020)

---

## 1. Executive Summary

The SolarWinds supply chain compromise, disclosed in December 2020, demonstrated advanced attacker tradecraft in establishing long-term persistence, stealthy command-and-control (C2), and durable control over enterprise environments.

Following initial access via a trojanized software update, the attackers leveraged identity-based techniques, stealthy beaconing mechanisms, and privilege escalation to maintain sustained access while evading detection.

The campaign highlights how attackers prioritize **persistence durability and controlled operations** over rapid exploitation, enabling prolonged access to high-value targets.

---

## 2. Threat Overview

| Category           | Details                             |
| ------------------ | ----------------------------------- |
| Type               | Advanced Persistent Threat (APT)    |
| Initial Vector     | Supply chain compromise             |
| Primary Capability | Stealth persistence and C2          |
| Impact             | Long-term espionage and data access |

---

## 3. Technical Analysis

### 3.1 Initial Access

The compromise originated from a malicious update to SolarWinds Orion software, which was distributed to multiple organizations as a trusted update.

This allowed attackers to execute code within privileged enterprise environments without triggering traditional security controls.

---

### 3.2 Persistence Mechanisms

Attackers established persistence through:

* Deployment of backdoored binaries within trusted applications
* Use of legitimate services to maintain execution
* Abuse of identity and privilege escalation to maintain access

Observed behavior includes:

* Persistence tied to legitimate software processes
* Avoidance of obvious registry or startup modifications
* Long dwell time without triggering alerts

This approach ensured persistence remained **low-noise and difficult to detect**.

---

### 3.3 Command-and-Control (C2)

The malware (SUNBURST) established a highly stealthy C2 channel.

Observed behavior includes:

* Delayed beaconing after initial infection
* Use of DNS-based communication
* Domain generation algorithms (DGA) for infrastructure flexibility
* Blending of malicious traffic with normal network activity

Unlike noisy beaconing, the communication was:

* Low frequency
* Environment-aware
* Designed to evade anomaly detection

---

### 3.4 Identity and Privilege Abuse

Following persistence, attackers:

* Escalated privileges within Active Directory
* Leveraged valid accounts for authentication
* Moved laterally using trusted protocols

Observed techniques:

* Abuse of federated identity systems
* Creation or compromise of privileged accounts
* Token manipulation and identity impersonation

This allowed attackers to operate as **legitimate users**, reducing detection likelihood.

---

### 3.5 Operational Control

Attackers maintained long-term control by:

* Rotating access mechanisms
* Maintaining multiple persistence layers
* Avoiding dependency on a single access vector

This ensured:

* Continued access even if part of the intrusion was detected
* Ability to re-enter environments after remediation attempts

---

## 4. Observed Indicators

### 4.1 Authentication Activity

* Use of legitimate credentials in abnormal contexts
* Privileged account usage without prior baseline
* Authentication originating from unusual hosts

---

### 4.2 Endpoint Activity

* Execution of trusted binaries with malicious modifications
* PowerShell and scripting activity tied to administrative operations

---

### 4.3 Network Activity

* Low-frequency outbound DNS communication
* Beaconing to attacker-controlled domains
* Traffic resembling legitimate application behaviour

---

### 4.4 Behavioral Indicators

* Long dwell time without detection
* Minimal noise in logs
* Gradual expansion of access rather than rapid propagation

---

## 5. Detection Opportunities

### Identity Monitoring

* Detection of anomalous privileged account usage
* Monitoring creation or modification of service accounts

---

### Endpoint Monitoring

* Unexpected execution of trusted applications
* PowerShell activity in non-administrative contexts

---

### Network Monitoring

* DNS beaconing patterns
* Low-frequency but consistent outbound connections

---

### Behavioral Detection

* Long-lived sessions with minimal activity
* Subtle deviations from baseline rather than spikes

---

## 6. Mitigation Strategies

* Implement strict monitoring of privileged accounts
* Enforce least privilege across service accounts
* Monitor for anomalous outbound DNS communication
* Enable enhanced logging (e.g., process creation, PowerShell)
* Validate integrity of software supply chains

---

## 7. MITRE ATT&CK Mapping

| Technique                         | ID    |
| --------------------------------- | ----- |
| Supply Chain Compromise           | T1195 |
| Valid Accounts                    | T1078 |
| Account Manipulation              | T1098 |
| Command and Scripting Interpreter | T1059 |
| Application Layer Protocol        | T1071 |
| Scheduled Task/Job                | T1053 |

Reference: MITRE ATT&CK Framework

---

## 8. Assessment

The SolarWinds campaign demonstrates that modern attackers prioritize **stealth, persistence, and identity control** over rapid exploitation.

Rather than relying on malware-heavy techniques, attackers:

* Blend into legitimate operations
* Leverage trusted identities
* Maintain multiple access paths

This aligns closely with enterprise intrusion patterns where detection is hindered by the use of valid credentials and low-noise communication.

---

## 9. Conclusion

The SolarWinds intrusion highlights the critical importance of monitoring identity activity, outbound communication, and long-term behavioural patterns.

Persistence and command-and-control are not isolated stages but are deeply integrated into attacker strategy, enabling sustained and controlled operations within compromised environments.

Organizations lacking visibility into identity behaviour and low-frequency network activity remain vulnerable to similar advanced threats.

---

## 10. References

* FireEye — "Highly Evasive Attacker Leverages SolarWinds Supply Chain"
* Microsoft — "Analyzing Solorigate, the compromised SolarWinds Orion software"
* CISA — "Alert AA20-352A: Advanced Persistent Threat Compromise of Government Agencies"
* MITRE ATT&CK Framework
