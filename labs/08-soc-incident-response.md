# Lab 08 — SOC: Incident Response & Containment

**Category:** Blue Team / Incident Response
**Environment:** TechBiz Security Academy — simulated practice range
**Context:** Direct continuation of the compromise from Lab 06 (r.malik's VPN account brute-forced and taken over).

## Objectives
- Review the endpoint process log
- Identify the persistence mechanism
- Identify the exfiltration destination
- Build the incident timeline
- Decide the containment action

## Methodology

### 1. Review the process log
```
logs
cat process.log
```
```
02:41:33 files01 r.malik   tar -czf /tmp/x.tgz /srv/finance
02:44:10 files01 r.malik   powershell -enc <base64 blob>
02:45:02 files01 SYSTEM    schtasks /create /tn "WinUpdateSvc" /tr svc.exe /sc onlogon
02:46:31 files01 SYSTEM    net user backup_adm P@ssw0rd! /add
02:46:40 files01 SYSTEM    net localgroup administrators backup_adm
```
This single log shows the full post-compromise chain: data staging → obfuscated PowerShell execution → scheduled-task persistence → rogue local admin account creation.

### 2. Identify the persistence mechanism
```
grep schtasks process.log
```
A scheduled task (`WinUpdateSvc`, disguised as a legitimate Windows update service) was created to run `svc.exe` on every logon — a standard, low-noise persistence technique on Windows.

### 3. Identify the exfiltration destination
```
cat exfil.log
```
```
02:52:18 files01 OUT 198.51.100.77:443  412 MB  duration 00:14:22
03:07:02 files01 OUT 198.51.100.77:443    2 MB  duration 00:00:11
```

### 4. Build the incident timeline
```
timeline
```
```
02:11  Password spray begins from 203.0.113.66
02:39  Successful authentication as r.malik
02:41  Finance directory archived to /tmp
02:45  Scheduled task created for persistence
02:46  Rogue local administrator account added
02:52  412 MB exfiltrated to 198.51.100.77
```

### 5. Decide the containment action
```
cat response.txt
contain
```
```
Order matters: isolate the host FIRST, preserve volatile evidence,
then remove persistence and the rogue account. Eradicating before
containment lets the attacker back in and destroys your timeline.
TBS{contain_before_eradicate}
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Windows persistence mechanism used | `scheduled task` |
| q2 | External IP that received the data | `198.51.100.77` |
| q3 | MB exfiltrated in the first transfer | `412` |
| q4 | NIST SP 800-61 phase isolating the host belongs to | `Containment` |
| q5 | Flag | `TBS{contain_before_eradicate}` |

## Takeaways
- Persistence (scheduled task) and privilege escalation (rogue local admin) both happened within ~5 minutes of initial compromise — dwell time before detection matters enormously.
- **Order of operations is critical in IR:** isolate/contain first, preserve volatile evidence, *then* eradicate. Removing malware or the rogue account before isolating the host risks tipping off the attacker (who may re-establish access) and destroys forensic artifacts needed for the timeline and root-cause analysis.
- This maps directly to the NIST SP 800-61 lifecycle: Preparation → Detection & Analysis → **Containment, Eradication & Recovery** → Post-Incident Activity. Host isolation sits squarely in the Containment sub-phase, which must precede Eradication.
