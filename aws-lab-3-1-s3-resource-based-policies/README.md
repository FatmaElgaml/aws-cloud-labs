# AWS Lab 3.1 — Securing an S3 Bucket with Identity-Based and Resource-Based Policies

Hands-on lab from the AWS Academy Cloud Foundations course. I signed in as a low-privilege IAM user, saw what its policy allows and denies, then used IAM roles and S3 bucket policies to gain access step by step.

**Result:** 100/100 (first submission 66.67 — I had skipped the Task 5 upload to bucket2, fixed it and resubmitted).

> Account ID and bucket names are redacted (`<ACCOUNT_ID>`, `<BUCKETn_NAME>`). This repo contains only my own notes, observations and screenshots, not the lab instructions.

## What I did

| Step | Action | Result |
|---|---|---|
| 1 | Signed in as `devuser` (IAM user, member of `DeveloperGroup`) | Console opens as devuser |
| 2 | Opened EC2 and S3 | EC2 denied (`ec2:DescribeInstances`, `ec2:DescribeKeyPairs`); S3 buckets visible, bucket1 objects listed |
| 3 | Read `DeveloperGroupPolicy` | Explains each result: no `ec2:*`, `s3:CreateBucket` + `s3:ListBucket` allowed, no `s3:GetObject`/`s3:PutObject` |
| 4 | Created a bucket, tried to upload | Bucket created; upload denied |
| 5 | Downloaded from bucket1 as devuser | `AccessDenied` for `s3:GetObject` |
| 6 | Switched to `BucketsAccessRole` | Download from bucket1 works; IAM pages denied (role has no IAM permissions) |
| 7 | Read the bucket2 bucket policy | It names `BucketsAccessRole`, so the role can upload to bucket2 even though its own policy only covers bucket1 |
| 8 | Challenge: upload to bucket3 | bucket3 trusts `OtherBucketAccessRole`; switching to it made the upload succeed |

## Key concepts

- **Identity-based policy:** attached to a user, group or role (`DeveloperGroupPolicy`, `GrantBucket1Access`).
- **Resource-based policy:** attached to the resource, here the S3 bucket policies on bucket2 and bucket3.
- **Role switching replaces permissions:** while a role is assumed, only the role's permissions apply.
- **Implicit deny and least privilege:** anything not allowed is denied, and the error message names the missing action.
- For a principal in the same account, one matching `Allow` (identity or resource policy) is enough unless something denies it.

## Repository contents

```
policies/      DeveloperGroupPolicy, GrantBucket1Access, bucket2-policy, bucket3-policy (sanitized)
screenshots/   18 numbered screenshots of the steps above (account ID and bucket names blacked out)
report/        Full lab report (Word) with figures and results table
```

## Skills practiced

AWS IAM (users, groups, roles, policies, trust), Amazon S3 bucket policies, STS role assumption, reading AWS access-denied errors, least-privilege thinking.
