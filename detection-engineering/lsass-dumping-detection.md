# LSASS-Dumping

## Technique Overview
LSASS dumping is a post-compromise credential access technique typically used during privilege escalation or lateral movement preparation. It is important to understand that at this stage, the attacker is in the infrastructure but have yet to fully control the infrastructure or execute on their objectives. Successful dumping significantly increases attacker operational capability by enabling credential reuse, impersonation, and persistence strategies.

LSASS dumping is important for attackers as LSASS provides attackers with access to credentials. 

## Telemetry Signal
Some of the telemetry signals includes...
* the use of Mimikatz, ProcDump exe 
* Sysmon log to see if process accessing LSASS memory
* unusual high privilege handle request
* suspicious dump file creation
* LSASS being accessed by Living-off-the-land binaries (rundll32, comsvcs) but normally doesn't access LSASS
* evidence of suspicious parent-child chain where
winword.exe → powershell.exe → rundll32.exe → lsass.exe


## Detection Logic
IF non-system process requests high-privilege access to LSASS AND parent process chain is uncommon AND dump artefact or credential tool behaviour observed → raise credential dumping alert.

## False Positives
Some non-system process can access LSASS.
* EDR and antivirus
* Backup and snapshot
* legitimate users (admins) troubleshooting

Instead of seeing LSASS dumping as an individual event, 
there are merits to see it as timeline. 

LSASS access + odd hours
LSASS access + new admin account
LSASS access + suspected beacon by NDR

would be higher risk


## How to see your own LSASS
- Go to Task Manager
- Right click lsass.exe, create dump file

## Intelligence Meaning 

Credential dumping activity targeting LSASS indicates an escalation in attacker capability and intrusion maturity. Successful extraction of authentication material enables lateral movement, privilege abuse, and potential long-term persistence. Analysts should pivot to authentication telemetry, service account usage, and anomalous administrative activity to assess campaign spread and operational depth.