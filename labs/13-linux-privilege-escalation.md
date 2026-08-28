# Lab 13 — Linux Privilege Escalation: From User to Root

**Category:** Post-Exploitation / Privilege Escalation
**Environment:** TechBiz Security Academy — simulated practice range
**Starting position:** Low-privilege shell as `student` on a compromised host.

## Objectives
- Check what sudo permits without a password
- Enumerate SUID binaries
- Review scheduled jobs
- Find a writable file that root executes
- Escalate to root
- Record the hardening advice

## Methodology

### 1. Check sudo rules
```
sudo -l
```
```
User student may run the following commands on files01:
(root) NOPASSWD: /usr/bin/find
```
`find` is a well-known GTFOBins entry — it supports `-exec`, which sudo will run as root with no password prompt.

### 2. Enumerate SUID binaries
```
suid
```
```
/usr/bin/passwd
/usr/bin/sudo
/bin/mount
/usr/bin/nmap  <-- unusual
```
`nmap` carrying the SUID bit is highly unusual and itself a separate escalation vector (older nmap versions support an interactive mode that can spawn a root shell) — flagged for hardening even though it wasn't the path exploited here.

### 3. Review scheduled jobs
```
crontab
```
```
*/5 * * * * root /opt/backup.sh
```
A root-owned cron job running every five minutes — worth checking the target script's permissions.

### 4. Find a writable file root executes
```
writable
```
```
/opt/backup.sh (root-owned job, world-writable: -rwxrwxrwx)
```
`backup.sh` is owned by root but has `777` permissions — any local user can edit it, and root will execute the edited contents on the next cron tick.

### 5. Escalate to root
```
exploit sudo-find
```
```
[+] Root shell obtained.
>> find can execute commands via -exec, and sudo runs it as root.
root@files01:~#
```
```
cat /root/proof.txt
```
```
root@files01
TBS{sudo_find_is_a_root_shell}
```

### 6. Record hardening advice
```
harden
```
```
- Remove NOPASSWD sudo rules for binaries that can spawn a shell (see GTFOBins).
- Strip the SUID bit from anything that does not genuinely require it.
- Root-run scripts must be root-owned and NOT world-writable.
- Keep the kernel and packages patched, but fix configuration first — it is the common path.
- Apply least privilege: every vector above is something granted more broadly than needed.
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Binary runnable as root without a password | `find` |
| q2 | Unusual binary carrying the SUID bit | `nmap` |
| q3 | Script root executes every five minutes | `backup.sh` |
| q4 | Single best hardening control | `least privilege` |
| q5 | Flag | `TBS{sudo_find_is_a_root_shell}` |

## Takeaways
- Root is almost never won through a novel exploit — it's almost always a misconfiguration: an overly permissive `sudo` rule, a needlessly SUID binary, or a privileged process trusting a file an unprivileged user can modify.
- [GTFOBins](https://gtfobins.github.io/) is the standard reference for which common Unix binaries can be abused for privilege escalation when granted excess sudo/SUID rights — checking `sudo -l` output and SUID listings against it should be a reflexive first step on any Linux box.
- A root-owned cron job pointing at a world-writable script is functionally identical to giving every local user root — the fix is simple and should be routine: `chmod 750` (or stricter) on any script executed by a privileged scheduler, and audit ownership/permissions on everything root touches on a schedule.
- Least privilege is the unifying theme across every vector found in this lab — each one is a case of a capability granted more broadly than the actual need required.
