# Persistence, Command-and-Control & Domain Control

---

## Campaign Context

This intelligence assessment reconstructs the third phase of a simulated enterprise intrusion focused on persistence establishment, command-and-control (C2) operations, and durable domain control through identity abuse.

Following successful credential access and lateral movement in Phase-2, the attacker transitions into maintaining long-term access and ensuring continued operational control over the compromised environment.

This phase demonstrates how attackers evolve from initial compromise to sustained control by leveraging identity mechanisms rather than software-based persistence.

---

## Phase Transition Overview

The attacker enters this phase with:

* Domain-level access on DC01
* Administrative privileges obtained via Pass-the-Hash
* Ability to execute commands across domain infrastructure

The objective shifts from access acquisition to **control durability and operational continuity**.

---

## Phase Objectives Observed

Adversary activity during this phase indicates intent to:

* Establish persistent access independent of initial compromise vector
* Maintain continuous remote command execution capability
* Ensure resilience against remediation efforts
* Expand control over identity infrastructure
* Transition from temporary access to durable domain dominance

These objectives reflect progression toward long-term operational control.

---

## Intrusion Activity Timeline Reconstruction

### 1. Persistence via Service Account Privilege Abuse

The attacker introduces a service-like account (`svc_backup`) and elevates it to Domain Administrator privileges.

Observed behavior includes:

* Creation or modification of service account
* Addition to Domain Admins group
* Privileged authentication using newly elevated identity

This enables persistence that is independent of the original compromised credentials.

---

### 2. Command-and-Control Establishment via PowerShell Beacon

The attacker deploys a PowerShell-based reverse beacon on the Domain Controller.

Observed behavior includes:

* Execution of PowerShell script (`beacon.ps1`)
* Outbound connection initiated from DC01 to attacker-controlled system
* Interactive command execution through established channel
* Scheduled task creation for recurring execution

This allows continuous remote control without requiring re-authentication.

---

### 3. Redundant Identity Persistence for Domain Control

To ensure resilience, the attacker creates an additional privileged account (`svc_ops`) and adds it to Domain Admins.

Observed behavior includes:

* Creation of secondary service account
* Elevation to Domain Administrator privileges
* Independent privileged access validation

This establishes multiple access paths, preventing complete eviction through partial remediation.

---

### 4. Resilience Against Defensive Actions

The attacker demonstrates that removal of a single privileged account does not eliminate access.

Observed behavior includes:

* Continued administrative control via secondary account
* Persistence of privileged access after account removal
* Multiple valid identities maintaining control

This reflects a shift from persistence to **durable domain control**.

---

## Intelligence Assessment

This phase demonstrates the transition from initial access to sustained operational control through identity-based techniques.

The attacker avoids traditional persistence mechanisms (e.g., malware, registry changes) and instead leverages:

* Legitimate account creation and modification
* Trusted administrative group membership
* Native system tools (PowerShell, scheduled tasks)

The combination of persistence, C2, and redundant identity control highlights a strategic approach focused on stealth, resilience, and long-term access.

This behavior is consistent with advanced intrusion campaigns where attackers prioritize control of the identity layer over host-level persistence mechanisms.

---

## Defensive Blind Spots Identified

* Limited monitoring of account creation and privilege escalation
* Lack of alerting on multiple Domain Admin accounts
* Insufficient visibility into PowerShell execution
* Absence of detection for scheduled task abuse
* Weak monitoring of outbound network connections from critical assets
* Lack of correlation between identity changes and network behaviour

These gaps enable attackers to maintain control without triggering high-confidence alerts.

---

## MITRE ATT&CK Mapping (Phase-3)

* **Account Manipulation (T1098)**
* **Valid Accounts (T1078)**
* **Create Account (T1136)**
* **Scheduled Task/Job (T1053)**
* **Command and Scripting Interpreter (T1059)**
* **Application Layer Protocol (T1071)**

---

## Phase Conclusion

The attacker has successfully transitioned from persistence to **active and durable domain control**, establishing multiple privileged identities and maintaining continuous command-and-control access.

This phase represents consolidation of control over the environment, where access is no longer dependent on a single vector and can withstand partial remediation efforts.

Without effective detection and response, the attacker is now positioned to proceed toward data exfiltration, impact operations, or further expansion of control across the enterprise environment.
