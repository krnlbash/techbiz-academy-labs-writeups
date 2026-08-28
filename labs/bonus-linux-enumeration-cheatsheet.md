# Bonus — Linux Enumeration Cheat Sheet

**Category:** Post-Exploitation / General Reference
**Context:** Practical enumeration commands run against a sandboxed Linux container, used as a repeatable checklist for CTFs and early-stage engagement recon.

## Checklist & Commands

### 1. Find where you are and list the files
```bash
pwd
ls -la
```
`pwd` prints the current working directory; `ls -la` lists all files (including hidden dotfiles) with permissions, owner, size, and timestamps.

### 2. Read a file with `cat`
```bash
cat filename.txt
cat -n file        # with line numbers
less file           # paginated view for long files
head -n 20 file     # first 20 lines
tail -n 20 file     # last 20 lines
```

### 3. Check who you are and your groups
```bash
whoami
id
groups
```
`whoami` gives the current username. `id` gives UID, GID, and all group memberships in one shot. `groups` lists just the group names.

### 4. Search the filesystem with `find`
```bash
find / -name "filename" 2>/dev/null
find / -type f -perm -4000 2>/dev/null    # SUID binaries
find / -newer /tmp/somefile 2>/dev/null   # files modified after a reference file
find / -type f -perm -o+w 2>/dev/null     # world-writable files
```
`2>/dev/null` suppresses "permission denied" noise from unreadable directories. `-type f`/`-type d` restrict to files/directories; `-perm` filters by permission bits.

### 5. Search inside files with `grep`
```bash
grep "pattern" filename
grep -r "pattern" /path/to/dir 2>/dev/null   # recursive
grep -rn "password" /etc 2>/dev/null         # recursive + line numbers
grep -i "pattern" file                       # case-insensitive
```

### 6. Inspect a file with unsafe permissions
```bash
ls -l filename
stat filename
```
`ls -l` shows the permission string (e.g. `-rwxrwxrwx`) and owner/group. `stat` gives more detail including octal mode and access/modify/change timestamps.

### 7. List running processes
```bash
ps aux
ps -ef
top       # or htop, for a live view
```
`ps aux` shows every process with user, PID, CPU/mem usage, and full command line — useful for spotting unexpected daemons or processes running with unusual privileges.

### 8. List listening network ports
```bash
ss -tulnp
netstat -tulnp   # older tool, may need net-tools installed
```
`-t` = TCP, `-u` = UDP, `-l` = listening only, `-n` = numeric (skip DNS lookups), `-p` = show the owning process (usually needs root for full visibility). If neither tool is installed, listening sockets can be read manually from `/proc/net/tcp` (hex-encoded local address/port, little-endian; state `0A` = `TCP_LISTEN`).

## Worked Example — Sandbox Container Enumeration

Running this checklist against an Ubuntu 24.04.4 LTS (Noble Numbat) sandbox container surfaced:

- **Identity:** running as `root` (`uid=0(root) gid=0(root)`), single group `root` — a root shell in a container carries a very different risk profile than root on a hypervisor host, worth noting in any writeup.
- **SUID binaries:** the expected standard set (`passwd`, `su`, `mount`, `umount`, `gpasswd`, `chsh`, `chfn`, `newgrp`, `polkit-agent-helper-1`, `dbus-daemon-launch-helper`) — nothing exotic like a stray SUID `python`/`bash`/`cp` that would signal a misconfiguration.
- **Permissions check:** `/etc/passwd` is `0644 root:root` (world-readable, expected). `/etc/shadow` is `0640 root:shadow` (correctly *not* world-readable). No world-writable files found under `/etc`. `/tmp` is `1777` (world-writable but sticky-bit protected — standard and safe).
- **Processes:** mostly kernel threads (`kworker`, `ksoftirqd`) plus the container's own init process — no unexpected user-space daemons.
- **Listening ports:** no `ss`/`netstat` binary installed, so `/proc/net/tcp` was decoded manually; found listeners on `2024` and `2025` bound to `0.0.0.0`, matching a `process_api` process seen in the process list, plus outbound established connections on `443` (HTTPS).

## Related finding: unsafe cron script permissions

While running this checklist during Lab 13 (Linux Privilege Escalation), `find` and `ls -l` surfaced `/opt/backup.sh` — root-owned but world-writable (`-rwxrwxrwx`, `777`) and executed by a root cron job every 5 minutes:

```bash
ls -l /opt/backup.sh
# -rwxrwxrwx 1 root root ... /opt/backup.sh
```

This is a textbook privilege-escalation vector: root-owned + world-writable + executed by a privileged scheduled process = any unprivileged local user can edit the script to run arbitrary commands as root on the next cron tick (e.g. adding themselves to `sudo`, spawning a reverse shell). See `13-linux-privilege-escalation.md` for the full exploitation chain.

**Fix:** cron scripts and any file executed by a privileged process should be root-owned *and* restrictively permissioned (`chmod 750` or stricter) — never world-writable.

## Takeaways
- This eight-step checklist (location → read → identity → search-by-name → search-by-content → permissions → processes → network) is a reliable, repeatable first pass on any newly obtained Linux shell, whether in a CTF, a real engagement, or routine system hardening review.
- The permission-inspection step (`ls -l`, `stat`, and the world-writable `find` sweep) is consistently the highest-value step for spotting privilege-escalation vectors, as demonstrated directly by the `backup.sh` finding above.
- Suppressing noise (`2>/dev/null`) matters in practice — unfiltered `find /` output against a full filesystem is often unusable without it.
