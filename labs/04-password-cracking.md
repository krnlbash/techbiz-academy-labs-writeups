# Lab 04 — Password Cracking: Hashes & Wordlists

**Category:** Credential Attacks
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- List the captured hashes
- Identify the hash algorithm
- Crack the admin hash
- Crack a second account
- Attempt the strong hash and observe the result

## Methodology

### 1. List captured hashes
```
hashes
```
```
admin   : 5f4dcc3b5aa765d61d8327deb882cf99
jsmith  : e10adc3949ba59abbe56e057f20f883e
finance : 25d55ad283aa400af464c76d713c07ad
```
Hashes carried over from the SQL injection lab (Lab 02).

### 2. Identify the algorithm
```
identify 5f4dcc3b5aa765d61d8327deb882cf99
```
```
Length      : 32 characters
Most likely : MD5
hashcat -m  : 0
```

### 3. Crack the admin hash
```
hashcat -m 0 5f4dcc3b5aa765d61d8327deb882cf99 rockyou.txt
```
```
5f4dcc3b5aa765d61d8327deb882cf99:password
```

### 4. Crack a second account
```
hashcat -m 0 e10adc3949ba59abbe56e057f20f883e rockyou.txt
```
```
e10adc3949ba59abbe56e057f20f883e:123456
Summary note: TBS{unsalted_md5_falls_in_seconds}
```

### 5. Attempt the strong hash (`finance`)
```
hashcat -m 0 25d55ad283aa400af464c76d713c07ad rockyou.txt
```
The `finance` account's hash resisted the wordlist — a longer/less-common password isn't present in `rockyou.txt`, illustrating that wordlist attacks are only as good as password strength and predictability, not the cracking speed of unsalted MD5.

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Algorithm | `MD5` |
| q2 | Hashcat mode number | `0` |
| q3 | Admin plaintext password | `password` |
| q4 | Control that would harden these hashes most | `salt` |
| q5 | Flag | `TBS{unsalted_md5_falls_in_seconds}` |

## Takeaways
- Unsalted MD5 is effectively a lookup problem: identical plaintexts always produce identical hashes, so common/weak passwords fall to a wordlist (or even a rainbow table) almost instantly.
- Salting each password individually defeats precomputed/rainbow-table attacks and forces per-hash cracking — it doesn't stop wordlist attacks against weak passwords outright, but it removes the ability to crack many accounts at once from a shared precomputed table.
- The `finance` account surviving the same wordlist shows that password *strength/uniqueness*, not hash strength alone, is what defeats a wordlist attack in practice — the two controls (salting, password strength) address different parts of the threat.
- Modern practice: use a slow, purpose-built password hash (bcrypt, scrypt, Argon2) with per-user salting, never fast general-purpose hashes like raw MD5/SHA1 for password storage.
