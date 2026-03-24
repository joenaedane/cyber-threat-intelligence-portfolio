# Unconstrained Delegation Attack Path

## Objective

This lab demonstrates how attackers enumerate and abuse **Unconstrained Kerberos Delegation** to discover privilege escalation paths that may lead to Domain Compromise.

---

## 🏗️ Lab Architecture

| Machine | Role                                                 |
| ------- | ---------------------------------------------------- |
| DC01    | Domain Controller                                    |
| WS01    | Low privilege workstation (corp\john)                |
| WS02    | Delegated machine (Unconstrained Delegation enabled) |
| Kali    | Attacker machine (BloodHound + Neo4j)                |

---

## Delegation Configuration (Performed on DC01)

Navigate to:

Active Directory Users and Computers → Computers → WS02 → Properties → Delegation Tab

Enable:

**Trust this computer for delegation to any service (Kerberos only)**

---

## Step 1 — Enumerate Delegated Machines

Run on WS01:

```powershell
Get-ADComputer -Filter {TrustedForDelegation -eq $True} -Properties TrustedForDelegation
```

### 📸 Screenshot

Insert:

```
screenshots/delegation-enumeration.png
```

### 🧠 Rationale

Unconstrained Delegation allows the host to store **user TGT in memory**.

If attacker later gains administrative access on this machine, they may steal tickets belonging to privileged users.

---

## 🌐 Step 2 — Validate Network Reachability

Commands:

```
ping ws02
net view \\ws02
```

### 📸 Screenshot

```
screenshots/reachability.png
```

### 🧠 Rationale

Attackers first confirm if lateral movement to target host is possible.

---

## 🧬 Step 3 — BloodHound Data Collection

Execute:

```
SharpHound.exe -c All
```

Transfer generated ZIP to Kali.

Import into BloodHound.

### 📸 Screenshot

```
screenshots/bloodhound-import.png
```

### 🧠 Rationale

BloodHound builds a graph of identity relationships including:

* Local admin rights
* Delegation
* Active sessions
* ACL abuse paths

---

## 🧭 Step 4 — Identify Attack Path to Delegated Host

Run BloodHound queries:

* Find Computers with Unconstrained Delegation
* Shortest Path from corp\john to WS02

### 📸 Screenshot

```
screenshots/attack-path.png
```

### 🧠 Rationale

Attackers prioritise delegated machines that are:

* reachable
* have privileged logons
* provide credential material

---

## 🧨 Step 5 — Privilege Escalation Reasoning

If attacker becomes local admin on WS02:

Possible actions:

* Dump LSASS memory
* Extract Kerberos TGT
* Perform Pass-the-Ticket
* Execute DCSync attack

This may lead to **full domain compromise.**

---

## 🛡️ Detection Opportunities

| Telemetry             | Reason                        |
| --------------------- | ----------------------------- |
| Event ID 4769 spike   | abnormal service ticket usage |
| Event ID 4624 Type 10 | admin logon to delegated host |
| LSASS access alerts   | credential dumping attempt    |
| Lateral movement logs | pivot behaviour               |

---

## 🔐 Mitigation Strategies

* Remove unconstrained delegation
* Use constrained or resource-based delegation
* Enforce privileged access workstations
* Monitor abnormal Kerberos activity

---

## 🎓 Key Learning Outcome

Delegation is not an exploit.

It becomes dangerous when chained with:

* Local admin rights
* Session exposure
* Credential dumping

Attackers succeed by **misconfiguration chaining.**

---
