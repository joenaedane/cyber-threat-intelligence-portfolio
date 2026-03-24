# # Active Directory Identity Attack Path Lab (BloodHound)

## Lab Objective

Analyse and prioritise Active Directory privilege escalation paths from the perspective of an attacker operating with compromised low-privilege domain credentials.

The lab focuses on identity attack graph modelling, Kerberos delegation abuse, and practical validation of escalation feasibility. Instead of demonstrating a single exploitation technique, the objective is to understand how multiple trust relationships in Active Directory can form realistic attack paths that enable lateral movement and privilege escalation.

This exercise mirrors real-world intrusion behaviour where attackers enumerate identity relationships, score potential escalation routes, and weaponise the most reliable path toward domain dominance.

---

## Lab Architecture

### Network Overview

```
VirtualBox Host-Only Network (192.168.56.0/24)

DC01   → 192.168.56.10  (Domain Controller / KDC / LDAP)
WS01   → 192.168.56.11  (User Workstation)
KALI   → 192.168.56.30  (Attacker Machine)
```

### Design Rationale

* Single flat subnet to simplify Kerberos communication
* Domain Controller provides authentication services
* Delegated server represents trusted application tier
* Attacker operates from compromised user context

---

## Lab Environment

| Component         | Version                                  |
| ----------------- | ---------------------------------------- |
| Domain Controller | Windows Server 2022                      |
| Workstation       | Windows 10                               |
| Attacker          | Kali Linux                               |
| Tools             | BloodHound, SharpHound, Impacket, Rubeus |

---

## Phase 1: Domain Controller Deployment and Set Up

### Steps

* Create VM DC001
* Configure static IP
* Install Active Directory Domain Services
* Promote forest `corp.local`
* Add John Smith in the AD Domain for WS01 to use
* Add WS01 into AD

### Screenshots

* ![alt text](./assets/identity-attack/image-1.png)
* ![alt text](./assets/identity-attack/image-2.png)
* ![alt text](./assets/identity-attack/image-4.png)
* ![alt text](./assets/identity-attack/image-5.png)
* ![alt text](./assets/identity-attack/image-6.png)

### Key Learning
Active Directory acts as centralized identity authority storing password hashes and Kerberos keys.

---

## Phase 2: Initial Access Simulation

### Assumption

Attacker obtained credentials:

```
corp\john / Password123!
```

### Actions

* Login to WS001
* Verify Kerberos ticket cache
* Discover domain controller

### Commands

```
whoami
klist
nltest /dsgetdc:corp.local
```

### Detection Opportunity

* Unusual login origin
* Kerberos TGT issuance patterns

### Screenshots
* ![alt text](./assets/identity-attack/image-7.png)

---

## Phase 3: Active Directory Enumeration

### Objective

Discover privilege escalation and delegation paths.

### Tool

SharpHound collection:

```
SharpHound.exe -c All
```

### Analysis

Import data into BloodHound and identify shortest path to Domain Admin.

### Screenshot Evidence

* ![alt text](./assets/identity-attack/image-8.png)

* ![alt text](./assets/identity-attack/image-9.png)

* ![alt text](./assets/identity-attack/image-10.png)

* ![alt text](./assets/identity-attack/image-11.png)

### Key Learning

- Enterprise identity is a graph of trust relationships exploitable by attackers.
- Session-based escalation paths require timing validation as attack graph relationships may not represent current exploit conditions.

- Initial enumeration revealed no deterministic privilege escalation paths from compromised user context. Escalation would require opportunistic session abuse, social engineering, credential harvesting, or configuration change exploitation.

## Engineered Privilege Misconfiguration

To simulate realistic enterprise misconfiguration, the compromised user account was granted **local administrative privileges** on workstation WS01.

Validation on host confirmed:
* ![alt text](./assets/identity-attack/image-12.png)


This introduced a deterministic escalation surface enabling:

- Administrative token acquisition
- Credential material access (LSASS memory)
- Persistence staging
- Lateral movement preparation

---

### Attack Graph Visibility Gap

Despite confirmed host-level privilege exposure, BloodHound graph ingestion did not display a corresponding `AdminTo` edge.

* ![alt text](./assets/identity-attack/image-13.png)

This highlights a critical operational insight:

> Attack graph accuracy depends on telemetry completeness and collection success.

Potential causes include:

- RPC/WMI collection failure
- Firewall restrictions
- UAC remote filtering
- Timing of enumeration
- Graph ingestion limitations

Therefore, **manual validation of privilege relationships is required** during attack path analysis.

---

### Attack Path Classification

| Path Type | Description | Reliability | Persistence | Impact |
|----------|------------|------------|------------|--------|
Session Abuse | Privileged session presence | Low | Transient | High |
Local Admin Exposure | Administrative control over WS01 | High | Persistent | Medium |

---

### Attacker Decision Rationale

An attacker would prioritise **persistent administrative foothold** over transient escalation opportunities.

Local administrative access enables:

- Credential harvesting
- Privilege discovery
- Token impersonation
- Subsequent lateral movement toward domain controllers

This staged escalation model reflects realistic adversary tradecraft.

---

## Detection Opportunities

### Local Privilege Manipulation

- Event ID 4732 – Member added to local Administrators group
- Event ID 4672 – Special privileges assigned to new logon
- Endpoint telemetry detecting privilege escalation patterns

### Credential Access Behaviour

- LSASS memory access detection
- Suspicious administrative token usage
- Abnormal SMB administrative share access

### Identity Attack Surface Monitoring

- Continuous privilege graph modelling
- Session presence monitoring on tier-0 assets
- Delegation and ACL exposure auditing

---
## Defensive Blind Spot Analysis

Attack path tooling may fail to represent real exploit conditions due to incomplete data collection.

Security teams must:

- Correlate host telemetry with identity graph analytics
- Validate privilege relationships operationally
- Avoid over-reliance on automated attack path visualisation

This lab demonstrates the importance of **human analytical reasoning in threat detection workflows.**

---

## Key Lessons Learned

- Privilege escalation opportunities may be conditional or persistent.
- Attack graph relationships require real-world validation.
- Administrative control over workstations enables staged domain compromise.
- Absence of Kerberoasting exposure does not imply secure identity posture.
- Attack path modelling is an iterative intelligence process.

---
