# Bonus — SMB Share Enumeration (Host Discovery to Data Read)

**Category:** Network Enumeration / SMB
**Target range:** `10.10.10.0/24`
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- Discover live hosts on the subnet
- Scan open ports on the file server
- Fingerprint service versions
- List SMB shares anonymously
- Read the world-readable share

## Methodology

### 1. Host discovery
```bash
nmap -sn 10.10.10.0/24 -oG - | awk '/Up$/{print $2}'
```
```
gateway (10.10.10.1)
web01   (10.10.10.5)
files01 (10.10.10.12)
db01    (10.10.10.20)
Nmap done: 256 IP addresses (4 hosts up) scanned
```
4 hosts live on the subnet. `files01` at `10.10.10.12` is identified as the file server by hostname.

> **Note:** if ICMP is filtered on a real range, an ARP scan is a more reliable fallback when L2-adjacent: `sudo arp-scan --interface=eth0 10.10.10.0/24`.

### 2. Port scan and service fingerprinting
```bash
nmap -sV -sC -p- --min-rate=1000 10.10.10.12 -oN fileserver_scan.txt
```
`-p-` scans all 65535 ports rather than assuming SMB only listens on 445 (some hosts also expose NetBIOS SMB on 139). `-sV -sC` grabs version banners and runs default NSE scripts (`smb-os-discovery`, `smb-protocols`), confirming SMB is listening on **port 445**.

### 3. Anonymous SMB share enumeration
```bash
smbclient -L //10.10.10.12/ -N
```
`-N` suppresses the password prompt for a null/anonymous session. (`enum4linux -a 10.10.10.12` is a useful fallback for a broader RID-cycling + share sweep if the null session is picky.) A share named `public` was found with read access for anonymous/Everyone.

### 4. Read the world-readable share
```bash
smbclient //10.10.10.12/public -N
smb: \> ls
```
```
notes.txt   A   162   Mon Aug 27 09:00:00 2026
```
```bash
smb: \> cat notes.txt
```
```
IT handover notes
-----------------
files01 still runs SMBv1 and is pending migration.
Do not store finance exports on this box.
TBS{null_session_exposes_share}
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Hosts live in `10.10.10.0/24` | `4` |
| q2 | IP address of the file server | `10.10.10.12` |
| q3 | Port SMB listens on | `445` |
| q4 | Name of the readable share | `public` |
| q5 | Flag | `TBS{null_session_exposes_share}` |

## Takeaways
- Anonymous/null SMB sessions on a legacy SMBv1 host let an unauthenticated attacker enumerate share names and read file contents with zero credentials — one of the oldest and still most common internal-network findings.
- A "public" share sitting alongside sensitive shares on the same unauthenticated listing is exactly the kind of misconfiguration that escalates quickly — the handover note itself flags that finance exports specifically should never land here.
- Remediation: restrict/disable null sessions (`RestrictAnonymous` on Windows), disable SMBv1 entirely, and apply least-privilege share permissions rather than relying on "nobody will look."
