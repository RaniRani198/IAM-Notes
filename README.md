# 🔐 AWS IAM MASTER NOTES — Zero to Hero  
A complete guide to mastering AWS Identity and Access Management (IAM), including debugging, permission boundaries, cross-account roles, S3 access, Glue/EMR roles, and real enterprise scenarios.

---

## 📚 Table of Contents
1. IAM Foundations  
2. IAM Policy Evaluation Logic  
3. Identity vs Resource Policies  
4. Trust Policies  
5. Permission Boundaries  
6. Service Control Policies (SCP)  
7. IAM Conditions  
8. IAM Debugging Framework (5-second method)  
9. IAM Cheatsheets  
10. Permission Boundary Master Guide  
11. S3, EMR, Glue IAM Requirements  
12. KMS Key Policy Essentials  
13. Top 20 IAM Real-World Scenarios  
14. Pocket Notes (Quick Revision)  

---

# 1️⃣ IAM FOUNDATIONS

### Identity-Based Policies  
Define **what the identity (user/role)** can do.

### Resource-Based Policies  
Define **who can access the resource** (S3, Lambda, SNS, SQS, Secrets Manager, KMS).

### Trust Policies  
Define **who is allowed to assume a role**.  
Critical for:
- Jenkins  
- EMR / EMR Serverless  
- Lambda  
- Glue  
- Cross-account roles  

### Permission Boundaries (PB)  
Restrict **maximum permissions** allowed for a user/role.

### Service Control Policies (SCP)  
Organization-level restricts → strongest DENY.

---

# 2️⃣ IAM POLICY EVALUATION LOGIC  
IAM always follows this order:

1. **Explicit Deny**  
2. **Explicit Allow**  
3. **Implicit Deny** (default)

This is the root of all IAM debugging.

---

# 3️⃣ IAM CONDITIONS  
The most powerful & most error-prone part.  
Examples:
- `aws:PrincipalArn`  
- `ArnsLike` / `StringEquals`  
- `s3:prefix`  
- `aws:RequestTag`  

Most hidden bugs come from wrong condition keys.

---

# 4️⃣ IAM DEBUGGING FRAMEWORK (5-second expert method)

### Step 1 — Identify Principal  
Which role/user made the call?

### Step 2 — Identify Action  
What EXACT API call failed?  
Example: `s3:ListBucket`, `glue:StartJobRun`, `sts:AssumeRole`

### Step 3 — Check Identity Policy  
Is the action allowed?

### Step 4 — Check Resource Policy  
Does the resource allow this principal?

### Step 5 — Check Trust Policy  
Is the assume-role principal correct?

### Step 6 — Check Permission Boundary  
PB may block the action even if identity allows.

### Step 7 — Check SCP  
Org-level restriction.

### Step 8 — Check Conditions  
Most subtle bugs appear here.

### Step 9 — Test in IAM Policy Simulator  

### Step 10 — Check CloudTrail Deny Reason  
**Final truth** for IAM.

---

# 5️⃣ IAM CHEATSHEETS

## 🟦 S3 Access Denied — Quick Fix
- Does identity allow `s3:ListBucket`?  
- Does identity allow `s3:GetObject`?  
- Does bucket policy include principal?  
- Does PB allow `s3:*` actions?  
- Does KMS key allow decrypt?  

---

## 🟪 AssumeRole Denied — Quick Fix
- Trust policy principal correct?  
- Identity allows `sts:AssumeRole`?  
- PB allows `sts:AssumeRole`?  
- SCP not blocking?  

---

## 🟧 Glue/EMR Failure — Quick Fix
Required IAM actions:
s3:GetObject
s3:PutObject
s3:DeleteObject
kms:Decrypt
iam:PassRole
glue:StartJobRun


---

# 6️⃣ PERMISSION BOUNDARY MASTER GUIDE

### Why PB Fails
PB sets MAXIMUM permissions.  
Identity allow cannot override PB deny.

### Best Practices
- Split PB into modules: S3, Glue/EMR, Jenkins, Core  
- Avoid wildcards (`iam:*`, `s3:*`)  
- Use flexible ARN matching:
     "ArnLike": { "aws:PrincipalArn": "arn:aws:iam:::role/svc.jenkins." }

  
---

# 7️⃣ S3, GLUE, EMR IAM REQUIREMENTS

### S3 Requires:
- `s3:ListBucket` on bucket  
- `s3:GetObject` on objects  
- `s3:PutObject` for writes  

### Glue Requires:
- `glue:StartJobRun`  
- `iam:PassRole`  
- S3 read/write  
- KMS decrypt  

### EMR Serverless Requires:
- S3 read/write  
- Logs bucket access  
- KMS key access  

---

# 8️⃣ KMS KEY POLICY MUST INCLUDE PRINCIPAL  
Even if IAM says allow, **KMS can still deny**.
"Principal": { "AWS": "arn:aws:iam::123456789012:role/MyRole" }


---

# 9️⃣ TOP 20 IAM REAL-WORLD SCENARIOS

### S3
1. Cannot list bucket  
2. Cannot read object  
3. Cannot write object  
4. Bucket deny overrides identity allow  
5. Incorrect prefix condition  

### AssumeRole
6. Jenkins cannot assume deployment role  
7. Cross-account assume-role fails  
8. Sandbox assume-role blocked  
9. Session policy mismatch  

### Permission Boundaries
10. PB blocks S3 PutObject  
11. PB missing Glue/EMR actions  
12. PB denies iam:PassRole  
13. PB too large (Checkov fail)  
14. PB denies EMR Serverless writes  

### Glue/EMR
15. Glue cannot write temp files  
16. EMR cannot write logs  
17. Glue JDBC failure  
18. EMR missing KMS decrypt  

### KMS
19. KMS decrypt failure  
20. KMS principal missing  

---

# 🔟 POCKET NOTES (Quick Revision)

- Explicit deny > allow  
- Missing allow = implicit deny  
- S3 always needs ListBucket + GetObject  
- KMS requires correct key policy  
- PB overrides identity policies  
- Expect failures from wrong ARN or wrong condition  

---


