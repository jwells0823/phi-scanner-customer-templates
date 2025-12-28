# PHI Cloud Monitor – Customer Role (CloudFormation) Setup

This CloudFormation template creates the **cross-account IAM role** and permissions that allow **PHI Cloud Monitor** to scan your AWS resources (starting with S3) without storing your data.

It is designed for the “vendor scans customer account” model:
- PHI Cloud Monitor runs the scanner in the **PHI Cloud Monitor main AWS account**
- The scanner **assumes a role** created in the **customer AWS account**
- The role grants **read-only** access needed for scanning (configurable by bucket scope)

---

## What this stack creates

In the **customer AWS account**, the stack typically creates:

- An IAM Role (ex: `PHICloudMonitorScannerRole`)
- A trust policy allowing PHI Cloud Monitor’s AWS account to assume that role
- An IAM policy granting read access required for scanning:
  - S3 bucket listing + object reads (optionally restricted to specific buckets/prefixes)
  - Optional KMS decrypt permissions if the bucket uses SSE-KMS (only if you enable it)

> Exact resources depend on the template you are deploying.

---

## Prerequisites

1. You have permission to deploy CloudFormation stacks in the customer account:
   - `cloudformation:CreateStack`, `iam:CreateRole`, `iam:PutRolePolicy`, etc.

2. You know the **PHI Cloud Monitor AWS Account ID** (the trusted vendor account).

Current PHI Cloud Monitor main account:
- **310022570453**

---

## Deploy via AWS Console (Recommended)

1. Log into the **customer AWS account**
2. Go to **CloudFormation → Stacks → Create stack → With new resources (standard)**
3. Under **Prepare template**, choose:
   - **Template is ready**
4. Under **Template source**, choose one:
   - **Upload a template file** (recommended)
   - or **Amazon S3 URL** (if you host it there)
5. Click **Next**

### Parameters
Fill in the parameters shown by the template. Common ones include:

- **TrustedAccountId**: `310022570453`
- **RoleName**: `PHICloudMonitorScannerRole` (or your preferred name)
- **BucketName** (optional): restrict permissions to a single bucket
- **BucketPrefix** (optional): restrict to a prefix (example: `phi/`)

> If you do not restrict bucket/prefix, the template may grant access to scan more broadly.
> For least privilege, restrict bucket and prefix where possible.

6. Click **Next**
7. On **Configure stack options**, keep defaults unless your org requires tags/policies
8. On **Review**, acknowledge IAM creation:
   - ✅ *“I acknowledge that AWS CloudFormation might create IAM resources…”*
9. Click **Create stack**

Wait until status shows: **CREATE_COMPLETE**

---

## Deploy via AWS CLI (Optional)

From a shell authenticated to the **customer account**:

```bash
aws cloudformation create-stack \
  --stack-name phi-cloud-monitor-scanner-role \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters \
    ParameterKey=TrustedAccountId,ParameterValue=310022570453 \
    ParameterKey=RoleName,ParameterValue=PHICloudMonitorScannerRole

