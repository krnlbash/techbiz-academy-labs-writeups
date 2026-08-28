# Lab 14 — Packet Analysis: Reading Traffic Like an Analyst

**Category:** Network Traffic Analysis
**Environment:** TechBiz Security Academy — simulated practice range
**Evidence:** `capture.pcap` from the incident

## Objectives
- Open the capture and read the summary
- Review the protocol breakdown
- Filter the HTTP traffic
- Follow the stream carrying the login
- Recover the cleartext credentials
- Spot the suspicious DNS activity

## Methodology

### 1. Capture summary and protocol breakdown
```
pcap
protocols
```
```
Packets  : 18,432
Duration : 00:22:14
Start    : 2026-08-26 02:38:11

TCP   12,881 (69.9%)
DNS    2,904 (15.8%)
HTTP   1,702 ( 9.2%)
FTP      604 ( 3.3%)
ICMP     341 ( 1.8%)
```

### 2. Filter HTTP traffic
```
filter http
```
```
GET  /login.php     HTTP/1.1  10.10.20.14 -> 10.10.10.5
POST /login.php     HTTP/1.1  10.10.20.14 -> 10.10.10.5 (302)
GET  /dashboard.php HTTP/1.1  10.10.20.14 -> 10.10.10.5
```

### 3. Follow the stream carrying the login
The credential-relevant conversation was actually on a separate stream (an FTP session, not the HTTP one filtered above):
```
follow 2
```
```
220 files01 FTP server ready
USER r.malik
331 Password required
PASS Summer2024!
230 Login successful
STOR payroll_2026.csv
226 Transfer complete
# note left on the server: TBS{cleartext_protocols_leak}
```

### 4. Recover cleartext credentials
```
creds
```
```
FTP 10.10.20.14 -> 10.10.10.12  user: r.malik  pass: Summer2024!
>> FTP transmits credentials in plaintext. Anyone on-path can read them.
```

### 5. Spot suspicious DNS activity
```
filter dns
```
```
A? update-cdn-node.net  (every 60s, 214 queries)
A? www.techbiz-sec.com
A? ntp.ubuntu.com
>> A domain queried on a fixed interval is classic C2 beaconing.
```
A domain being queried at a precise, fixed 60-second interval (214 times over the capture) is a textbook C2 beaconing pattern — legitimate DNS lookups don't repeat with clockwork regularity.

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Cleartext protocol that leaked the password | `FTP` |
| q2 | Password captured in cleartext | `Summer2024!` |
| q3 | Domain the beaconing is going to | `update-cdn-node.net` |
| q4 | Single change that would have prevented the leak | `encryption` |
| q5 | Flag | `TBS{cleartext_protocols_leak}` |

## Takeaways
- FTP (and other legacy plaintext protocols — Telnet, unencrypted HTTP, POP3/IMAP without TLS) send credentials in the clear over the wire. Anyone with visibility into the traffic path — a compromised switch, a rogue access point, an on-path attacker — can trivially recover them with nothing more than a packet capture.
- The fix is simple and non-negotiable: replace plaintext protocols with their encrypted equivalents (SFTP/FTPS instead of FTP, HTTPS instead of HTTP, IMAPS/POP3S instead of unencrypted mail).
- Fixed-interval DNS queries to an unfamiliar domain are one of the most reliable low-level indicators of C2 beaconing — malware implants commonly "call home" on a set schedule for tasking, and that regularity stands out clearly against normal, bursty user-driven DNS traffic once you look at query timing rather than just query volume.
