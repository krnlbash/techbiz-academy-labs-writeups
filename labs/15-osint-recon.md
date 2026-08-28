# Lab 15 — OSINT: Mapping a Target Before You Touch It

**Category:** Open-Source Intelligence / Passive Reconnaissance
**Target:** `example-corp.com` (simulated dataset)
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- Pull the registration record
- Enumerate DNS records
- Discover subdomains passively
- Determine the email address format
- Check an address against breach data
- Find the exposed repository

## Methodology

### 1. Registration record
```
whois example-corp.com
```
```
Registrar     : Example Registrar Inc.
Created       : 2011-04-18
Expires       : 2027-04-18
Registrant    : Example Corp (privacy protected)
Name servers  : ns1/ns2.example-corp.com
```

### 2. DNS records
```
dns example-corp.com
```
```
A   example-corp.com          203.0.113.10
MX  example-corp.com          mail.example-corp.com
TXT example-corp.com          "v=spf1 include:_spf.example.com ~all"
>> SPF present. DKIM selector found. NO DMARC record — spoofing is easier without it.
```
An SPF record without a corresponding DMARC record means there's no policy telling receiving mail servers what to do with messages that fail SPF/DKIM alignment — making the domain considerably easier to spoof for phishing.

### 3. Passive subdomain discovery
```
subdomains example-corp.com
```
```
www.example-corp.com      203.0.113.10
mail.example-corp.com     203.0.113.11
vpn.example-corp.com      203.0.113.12
dev.example-corp.com      203.0.113.44   <-- non-production
legacy.example-corp.com   203.0.113.45
```
`dev.example-corp.com` stands out as a forgotten/non-production environment — these are frequently under-hardened compared to production and a common entry point.

### 4. Email address format
```
emails example-corp.com
```
```
j.smith@example-corp.com
a.khan@example-corp.com
r.malik@example-corp.com
>> Format is first.lastname — enough to guess any employee address.
```
Once the naming convention is known, any employee's corporate email can be predicted from their public name alone (e.g. from LinkedIn) — useful for both phishing simulation and legitimate red-team targeting.

### 5. Breach data check
```
breaches j.smith@example-corp.com
```
```
Found in 2 historical breach corpora (2019, 2021).
>> Reused passwords are how credential stuffing succeeds.
```

### 6. Exposed repository
```
leaks
```
```
Public repository: example-corp/internal-scripts
deploy.sh committed 2024-11-02 — contains a hardcoded API key (since rotated)
README.md references vpn.example-corp.com and the dev environment
TBS{public_data_is_attack_surface}
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Organisation email format | `first.last` |
| q2 | Subdomain that looks like a forgotten test environment | `dev.example-corp.com` |
| q3 | Mail security record missing from DNS | `DMARC` |
| q4 | Why passive OSINT is legally safer than active scanning | `no contact` (queries only third-party public records — never touches the target's own systems) |
| q5 | Flag | `TBS{public_data_is_attack_surface}` |

## Takeaways
- Passive reconnaissance (WHOIS, DNS records, certificate transparency logs, public code repositories, breach-data aggregators) builds a substantial attack surface picture without ever sending a single packet to the target's own infrastructure — which is precisely why it doesn't require prior authorization the way active scanning does.
- A missing DMARC record, even with SPF/DKIM present, leaves a meaningful spoofing gap — all three should be configured together (SPF + DKIM + DMARC with an enforcement policy) for real protection.
- Forgotten subdomains (`dev.`, `legacy.`, `staging.`) discovered via passive sources are consistently one of the highest-value findings in real engagements — they're frequently unpatched, unmonitored, and running outdated software compared to production.
- Public code repositories are a recurring source of hardcoded secrets, internal hostnames, and infrastructure details — even a single old commit (`deploy.sh` here) can leak information long after the "current" state of a repo looks clean.
