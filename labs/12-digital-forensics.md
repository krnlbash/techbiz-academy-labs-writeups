# Lab 12 — Digital Forensics: Reconstructing What Happened

**Category:** Digital Forensics & Incident Response (DFIR)
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- Review the evidence details
- Verify the acquisition hash
- List files including deleted entries
- Recover the deleted file
- Build the artefact timeline

## Methodology

### 1. Review evidence details
```
image
```
```
Image    : files01.dd
Size     : 64 GB
Acquired : 2026-08-26 11:04 UTC
Examiner : SOC-2
SHA256   : 9f2c...a11e
```

### 2. Verify acquisition integrity
```
verify
```
```
Acquisition : 9f2c...a11e
Recomputed  : 9f2c...a11e
[+] MATCH — evidence integrity intact.
```
Recomputing and comparing the hash proves the image has not been altered since acquisition — this is what makes the evidence defensible/admissible; a mismatch would mean the chain of custody is broken.

### 3. List files including deleted entries
```
fls
```
```
notes.txt
svc.exe
* x.tgz
prefetch.db
>> Entries marked * are deleted but still recoverable.
```
File system metadata (the inode/MFT entry) for `x.tgz` still exists even though it was deleted — the entry is simply marked as unallocated, not wiped.

### 4. Recover the deleted file
```
recover x.tgz
icat x.tgz
```
```
Recovered archive listing:
 /srv/finance/payroll_2026.csv
 /srv/finance/vendors.csv
Persistence artefact: HKCU\...\Run\WinUpdateSvc
TBS{deleted_is_not_gone}
```
Because the underlying disk blocks were never overwritten by new data, the file carves cleanly from unallocated space — confirming both the staged finance data and the registry Run-key persistence artefact.

### 5. Build the artefact timeline
```
timeline
```
```
02:41  x.tgz created in /tmp (finance data staged)
02:45  svc.exe written to disk
02:45  Run key WinUpdateSvc created (persistence)
02:52  Outbound transfer begins
03:14  x.tgz deleted — but the blocks were never overwritten
11:04  Image acquired, hash recorded
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Does the acquisition hash match the image | `yes` |
| q2 | Filename of the deleted archive | `x.tgz` |
| q3 | Registry-style key that gave persistence | `Run` |
| q4 | Why the image must be hashed before analysis | `integrity` |
| q5 | Flag | `TBS{deleted_is_not_gone}` |

## Takeaways
- Hashing the acquired image immediately and re-verifying before analysis is a foundational forensic principle: it proves the evidence hasn't been tampered with and preserves chain-of-custody defensibility, especially if findings ever need to hold up legally.
- "Deleting" a file on most filesystems only removes the directory/metadata pointer to it — the actual data blocks remain on disk (in unallocated space) until overwritten by new writes. This is why deleted files are frequently recoverable in forensic investigations, sometimes long after deletion.
- Building a unified artefact timeline (correlating process execution, persistence creation, exfiltration, and deletion events) is what turns isolated findings into a coherent incident narrative — essential for both root-cause analysis and any legal/HR follow-up.
