# TechBiz Security Academy — Lab Writeups

Writeups for the [TechBiz Security Academy](https://academy.techbiz-sec.com/) practice range — a simulated, self-contained lab environment covering web application security, offensive tooling, blue-team/SOC work, cloud security, and digital forensics.

> All labs were completed in a fully simulated environment. No real systems were contacted, scanned, or exploited at any point.

## About

Each writeup follows the same structure:
- **Objectives** — what the lab asked for
- **Methodology** — the exact commands run and the reasoning behind each step
- **Answers** — the graded question set with final answers
- **Takeaways** — the underlying security concept and real-world relevance, beyond just "the answer"

## Index

| # | Lab | Category | Flag |
|---|---|---|---|
| 01 | [Web Enumeration & Hidden Content](labs/01-web-enumeration.md) | Web / Recon | `TBS{backup_files_leak_creds}` |
| 02 | [SQL Injection: From Error to Data](labs/02-sql-injection.md) | Web / Database | `TBS{union_select_dumps_the_db}` |
| 03 | [Cross-Site Scripting: Filters & Bypasses](labs/03-xss-filters-bypasses.md) | Web / Client-side | `TBS{attribute_break_out_xss}` |
| 04 | [Password Cracking: Hashes & Wordlists](labs/04-password-cracking.md) | Credential Attacks | `TBS{unsalted_md5_falls_in_seconds}` |
| 05 | [EternalBlue (MS17-010)](labs/05-eternalblue-ms17-010.md) | Exploitation / SMB | `TBS{ms17_010_system_shell}` |
| 06 | [SOC: Log Triage & Brute-Force Detection](labs/06-soc-log-triage.md) | Blue Team / SOC | `TBS{brute_force_then_success}` |
| 07 | [SOC: Phishing Email Analysis](labs/07-soc-phishing-analysis.md) | Blue Team / Email Security | `TBS{spf_fail_lookalike_domain}` |
| 08 | [SOC: Incident Response & Containment](labs/08-soc-incident-response.md) | Blue Team / IR | `TBS{contain_before_eradicate}` |
| 09 | [Active Directory: Path to Domain Admin](labs/09-active-directory-domain-admin.md) | Active Directory | `TBS{spn_to_domain_admin}` |
| 10 | [Cloud Misconfiguration: Buckets, Roles & Metadata](labs/10-cloud-misconfiguration.md) | Cloud Security / AWS | `TBS{public_bucket_to_role_takeover}` |
| 11 | [Malware Triage: Static Analysis](labs/11-malware-triage.md) | Malware Analysis | `TBS{never_trust_the_extension}` |
| 12 | [Digital Forensics: Reconstructing What Happened](labs/12-digital-forensics.md) | DFIR | `TBS{deleted_is_not_gone}` |
| 13 | [Linux Privilege Escalation](labs/13-linux-privilege-escalation.md) | Post-Exploitation | `TBS{sudo_find_is_a_root_shell}` |
| 14 | [Packet Analysis: Reading Traffic Like an Analyst](labs/14-packet-analysis.md) | Network Traffic Analysis | `TBS{cleartext_protocols_leak}` |
| 15 | [OSINT: Mapping a Target Before You Touch It](labs/15-osint-recon.md) | OSINT / Passive Recon | `TBS{public_data_is_attack_surface}` |
| 16 | [Cryptography & Encoding: Telling Them Apart](labs/16-cryptography-encoding.md) | Cryptography Fundamentals | `TBS{encoding_is_not_encryption}` |
| Bonus | [SMB Share Enumeration](labs/bonus-smb-share-enumeration.md) | Network Enumeration | `TBS{null_session_exposes_share}` |
| Bonus | [Linux Enumeration Cheat Sheet](labs/bonus-linux-enumeration-cheatsheet.md) | Reference / Post-Exploitation | — |

## Topics Covered

- **Web Application Security** — enumeration, SQL injection, cross-site scripting, secure headers
- **Credential Attacks** — hash identification, cracking, salting, Kerberoasting
- **Exploitation** — MS17-010/EternalBlue, Active Directory attack path mapping
- **Blue Team / SOC** — log triage, brute-force detection, phishing analysis, incident response ordering (NIST SP 800-61)
- **Cloud Security** — public bucket exposure, IMDS/SSRF credential theft, IAM least privilege
- **Malware Analysis** — static triage, PE import analysis, threat intel correlation
- **Digital Forensics** — evidence integrity, deleted-file recovery, timeline reconstruction
- **Linux Privilege Escalation** — sudo misconfiguration, SUID binaries, writable root-executed scripts
- **Network Traffic Analysis** — protocol identification, credential leakage, C2 beaconing detection
- **OSINT** — passive reconnaissance, DNS/email enumeration, breach data, exposed repositories
- **Cryptography Fundamentals** — encoding vs. encryption vs. hashing

## Disclaimer

These writeups document work performed exclusively against a simulated, sandboxed practice range built for training purposes. Only ever test systems you own or are explicitly authorised in writing to test.

---

*Maaz — [krnlbash](https://github.com/) · CTF team 0x141*
