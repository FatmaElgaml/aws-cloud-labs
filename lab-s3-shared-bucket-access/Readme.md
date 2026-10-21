# Securing a Shared Amazon S3 Bucket

**AWS Hands-on Lab | Security & IAM**

This lab was completed with Abdo as part of our AWS technical training.

The goal was to secure an Amazon S3 bucket shared by a data team. Each team member needed access to specific folders, team leads needed a few extra permissions, and some files had to remain completely private.

We also needed to make one folder publicly accessible without exposing the rest of the bucket.

## 1. The Scenario

We used an S3 bucket in the Ireland region (`eu-west-1`) to simulate a company's shared data storage.

The bucket contains five folders, each with different access rules:

| Folder | Team Members | Team Leads | Public Visitors |
|---|---|---|---|
| `shared/` | Read and upload | Read and upload | No access |
| `reports/` | Read only | Read and publish | No access |
| `public/` | Read only | Read and publish | Read specific files |
| `home/<username>/` | Access to their own folder only | Access to their own folder only | No access |
| `restricted/` | No access | No access | No access |

The main challenge was to apply these rules without giving users unnecessary permissions or exposing private files.

## 2. What We Set Up

### S3 Bucket

We created a general-purpose S3 bucket and uploaded sample files to test the different access rules.

We kept S3 Block Public Access enabled initially, disabled ACLs, and used SSE-S3 for server-side encryption.

### IAM Users and Groups

We created two IAM groups:

- `DataTeam-Members`
- `DataTeam-Leads`

The bucket access was managed through two customer-managed IAM policies, attached to the groups rather than directly to individual users.

We created two team members and one team lead. The lead belongs to both groups, so they receive the basic team permissions as well as the additional publishing permissions.

All users had console access only, with no access keys created.

### Access Policies

We used two IAM policies to control what each group could do:

- **`DataTeam-S3-Base`** gives team members access to shared files, read-only access to reports, and access to their own home folder.
- **`DataTeam-S3-Publish`** gives team leads permission to upload and delete files in the reports and public folders.

The `restricted/` folder was left out of all Allow statements. This means users cannot access it through these policies.

### Making the Public Folder Accessible

We wanted visitors to open published files without making the entire bucket public.

To achieve this, we adjusted the relevant S3 Block Public Access settings and added a bucket policy that allows anonymous users to read objects inside `public/`.

We also added an explicit Deny for requests that do not use HTTPS.

This allows visitors to open a published file through its direct HTTPS URL, but it does not allow them to browse the folder or access other private files.

## 3. Testing the Permissions

After configuring the bucket and IAM policies, we tested the setup using separate private browser windows for different users.

We also tested anonymous access while signed out of AWS.

Here are some of the results:

| Test | Result |
|---|---|
| Team member downloads files from `shared/` and `reports/` | Allowed |
| Team member creates an object in their own home folder | Allowed |
| Team member accesses another user's home folder | Denied |
| Team member uploads to `reports/` or `public/` | Denied |
| Team member accesses `restricted/` | Denied |
| Team lead uploads files to `reports/` and `public/` | Allowed |
| Anonymous visitor opens `public/index.html` over HTTPS | Allowed |
| Anonymous visitor opens files in `shared/` or `restricted/` | Denied |
| Anonymous visitor tries to list the `public/` folder | Denied |
| Anonymous visitor requests a public file over HTTP | Denied |

For the HTTP test, we used `curl.exe -i` because browsers can automatically upgrade HTTP requests to HTTPS, which could hide whether our HTTPS-only rule was working.

We planned 18 tests across the different personas and the IAM Policy Simulator. At the time of writing, 14 tests had screenshot evidence. Some additional tests had been set up but were not fully documented with screenshots.

The detailed report and available evidence are included in the `report/` directory.

## 4. What I Learned

This lab helped me understand how S3 security works in practice, especially when multiple users share the same bucket.

A few things stood out:

- **Manage permissions through groups.** When a new employee joins, adding them to the right group is easier than assigning policies individually.
- **Keep private folders private by default.** If a policy does not grant access, the request is denied unless another applicable policy grants it.
- **Use the correct resource ARN.** Listing a bucket requires permissions on the bucket ARN, while reading or uploading files requires permissions on the relevant object ARNs.
- **Use policy variables for personal folders.** The `${aws:username}` variable lets one IAM policy give each user access to their own home folder.
- **Be careful when enabling public access.** Public access should be limited to the exact objects that need it, rather than the entire bucket.
- **Public file access does not require public folder listing.** Visitors can open a file through its direct URL without receiving permission to list the folder.
- **Explicit Deny takes precedence.** The HTTPS-only rule blocks plain HTTP requests even if another applicable policy allows the requested action.
- **Test permissions from different perspectives.** An administrator's successful access does not prove that a regular user or an anonymous visitor has the correct permissions.

## 5. Repository Structure

```text
.
├── policies/
│   ├── DataTeam-S3-Base.json
│   ├── DataTeam-S3-Publish.json
│   └── bucket-policy.json
├── sample-data/
└── report/
    └── S3_Shared_Bucket_Lab_Report_redacted.pdf
```

The repository contains sanitized policy files, sample files with fake data, and a report with the account ID redacted.

## Final Thoughts

This lab was a practical way to understand how IAM policies and S3 bucket policies work together.

Instead of giving everyone broad access to the bucket, we used group-based permissions, private user folders, and a narrowly scoped public folder to control access.

It also showed me why testing matters just as much as configuring the policies. A setup is not complete until we verify both what users should be able to access and what they should not.

**Completed with Abdo(amra88189-arch) | AWS Technical Training**
