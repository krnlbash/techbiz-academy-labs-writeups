# Lab 06 — SOC: Log Triage & Brute-Force Detection

**Category:** Blue Team / SOC
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- List and open the authentication log
- Count the failed authentication attempts
- Identify the attacking source IP
- Find the successful login that followed
- Record the verdict

## Methodology

### 1. Review the log
```
logs
cat auth.log
```
```
2026-08-26 02:11:04 vpn01 sshd: FAILED login for admin from 203.0.113.66
2026-08-26 02:11:06 vpn01 sshd: FAILED login for administrator from 203.0.113.66
... 43 further FAILED entries from 203.0.113.66 (02:11:09 - 02:38:52) ...
2026-08-26 02:38:55 vpn01 sshd: FAILED login for r.malik from 203.0.113.66
2026-08-26 02:39:01 vpn01 sshd: SUCCESS login for r.malik from 203.0.113.66
2026-08-26 02:39:14 vpn01 sshd: session opened for user r.malik
2026-08-26 02:41:33 vpn01 sudo: r.malik : COMMAND=/usr/bin/tar -czf /tmp/x.tgz /srv/finance
2026-08-26 09:02:11 vpn01 sshd: SUCCESS login for a.khan from 10.10.20.14
```

### 2. Count failures and confirm source
```
count FAILED auth.log
grep 203.0.113.66 auth.log
```
```
47 matching lines
```
All 47 FAILED entries originate from the single IP `203.0.113.66` — consistent with a scripted password-spray attack.

### 3. Confirm the successful login
```
grep SUCCESS auth.log
```
```
SUCCESS login for r.malik from 203.0.113.66
SUCCESS login for a.khan from 10.10.20.14
```
`r.malik` authenticated successfully from the *same* attacking IP immediately after the failed spray — a clear account compromise. (`a.khan`'s success from an internal IP is unrelated, legitimate activity.)

### 4. Record the verdict
```
cat incident.txt
verdict
```
```
Password spray followed by a successful authentication.
Account r.malik confirmed compromised. No MFA on the VPN group.
TBS{brute_force_then_success}
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Failed logins from the attacking IP | `47` |
| q2 | Attacking source IP | `203.0.113.66` |
| q3 | Account ultimately compromised | `r.malik` |
| q4 | Control that would have stopped this outright | `MFA` |
| q5 | Flag | `TBS{brute_force_then_success}` |

## Takeaways
- A high volume of `FAILED` entries from a single source IP against multiple usernames (`admin`, `administrator`, `root`...) followed by a `SUCCESS` is the textbook signature of a password spray / brute-force that eventually landed.
- The immediately-following `sudo tar` command against `/srv/finance` shows the attacker moved straight to data staging post-compromise — worth flagging in the timeline for scope of impact.
- MFA is the single control that would have stopped this outright: even with a correctly guessed password, the login would fail without the second factor.
