# Lab 07 — SOC: Phishing Email Analysis

**Category:** Blue Team / Email Security
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- Open the reported email
- Inspect the authentication results
- Identify the real sending domain
- Extract the payload link
- Record the verdict

## Methodology

### 1. Open the reported email
```
logs
cat email.eml
```
```
Received: from mail.techbiz-secure-login.com (203.0.113.201)
Authentication-Results: spf=fail (sender IP is 203.0.113.201);
 dkim=none; dmarc=fail action=quarantine
From: "TechBiz IT Support" <it-support@techbiz-secure-login.com>
To: a.khan@techbiz-sec.com
Subject: [URGENT] Your mailbox will be disabled in 24 hours

Dear user,
Our records show your mailbox quota is exceeded. Verify within 24 hours
or access will be permanently revoked.

<a href="https://techbiz-secure-login.com/owa/verify?id=8842">https://mail.techbiz-sec.com/owa</a>

IT Support Desk
```

### 2. Inspect authentication results
```
grep Authentication-Results email.eml
```
```
spf=fail (sender IP is 203.0.113.201); dkim=none; dmarc=fail action=quarantine
```
SPF fails, DKIM is absent, and DMARC fails — none of the three core email-authentication mechanisms pass.

### 3. Identify the real sending domain / lookalike
The `From` display name reads "TechBiz IT Support" but the actual address (`it-support@techbiz-secure-login.com`) and the `Received` header both resolve to a **lookalike domain**, `techbiz-secure-login.com` — registered to imitate the legitimate `techbiz-sec.com`.

### 4. Extract the real payload link
```
grep href email.eml
```
The visible link text reads `https://mail.techbiz-sec.com/owa` (the legitimate-looking domain), but the actual `href` target is:
```
https://techbiz-secure-login.com/owa/verify?id=8842
```
Classic link-text/href mismatch — the anchor text is spoofed to build trust while the real destination is the attacker's lookalike domain.

### 5. Record the verdict
```
cat analysis.txt
verdict
```
```
SPF fail, DKIM absent, DMARC fail. Lookalike domain registered 6 days ago.
Display name impersonates internal IT. Link text does not match the href.
Verdict: malicious credential phishing. Block domain, purge, reset if clicked.
TBS{spf_fail_lookalike_domain}
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Did SPF pass or fail | `fail` |
| q2 | Domain the link actually points to | `techbiz-secure-login.com` |
| q3 | Technique of faking the From display name | `spoofing` |
| q4 | First action for other recipients | `block` (block sender/domain org-wide before purge/reset) |
| q5 | Flag | `TBS{spf_fail_lookalike_domain}` |

## Takeaways
- All three authentication mechanisms failing simultaneously (SPF fail, no DKIM, DMARC fail) is an extremely strong phishing signal on its own, before even reading the body.
- Domain age (6 days old) combined with a lookalike hostname is a classic newly-registered-domain (NRD) indicator commonly used in threat intel scoring.
- Always compare visible link text against the actual `href` target — display text is attacker-controlled and proves nothing about the real destination.
- Response order matters: block/quarantine the sending domain org-wide first to protect other recipients, then purge delivered copies, then reset credentials for anyone confirmed to have clicked.
