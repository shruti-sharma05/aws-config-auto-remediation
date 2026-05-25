# Automated Cloud Security Posture Management on AWS

## 📌 Project Overview
This project demonstrates automated cloud security monitoring and remediation using AWS native services. When an S3 bucket is misconfigured as publicly accessible, the system automatically detects the violation and remediates it — without any manual intervention.

---

## 🛠️ AWS Services Used

| Service | Purpose |
|---|---|
| AWS CloudTrail | Logs all API activity for auditing |
| AWS Config | Continuously monitors resource configurations |
| Amazon EventBridge | Captures compliance change events and triggers automation |
| AWS Lambda | Executes auto-remediation logic |
| Amazon S3 | The resource being monitored and secured |
| Amazon CloudWatch | Stores Lambda execution logs |

---

## ⚙️ Architecture Flow

```
AWS Config (detects noncompliant S3 bucket)
        │
        ▼
Amazon EventBridge (rule: NON_COMPLIANT trigger)
        │
        ▼
AWS Lambda (auto-remediate-s3)
        │
        ▼
Amazon S3 (Block Public Access enabled)
        │
        ▼
CloudWatch Logs (execution logged)
```

---

## 🔄 How It Works

1. **AWS Config** continuously monitors S3 bucket configurations using managed rules:
   - `s3-bucket-public-read-prohibited`
   - `s3-bucket-public-read-write-prohibited`
   - `restricted-ssh`
   - `root-account-mfa-enabled`

2. When a bucket is found **noncompliant** (public access enabled), Config flags it.

3. **Amazon EventBridge** captures the compliance change event using this pattern:
   ```json
   {
     "source": ["aws.config"],
     "detail-type": ["Config Rules Compliance Change"],
     "detail": {
       "newEvaluationResult": {
         "complianceType": ["NON_COMPLIANT"]
       }
     }
   }
   ```

4. EventBridge triggers the **Lambda function** (`auto-remediate-s3`).

5. Lambda calls `put_public_access_block` on the specific noncompliant bucket, blocking all public access.

6. **CloudWatch Logs** records the execution for auditing.

---

## 📁 Project Structure

```
├── README.md                  # Project documentation
├── lambda_function.py         # Lambda auto-remediation code
├── eventbridge-rule.json      # EventBridge event pattern
├── iam-policy.json            # IAM policy for Lambda execution role
└── screenshots/               # AWS Console screenshots
    ├── compliance-status-after.png
    ├── config-noncompliant-bucket.png
    ├── eventbridge-event-source.png
    ├── eventbridge-target.png
    ├── cloudtrail-trails.png
    ├── config-dashboard-before.png
    ├── config-rules.png
    ├── s3-bucket-permissions.png
    ├── lambda-function.png
    └── eventbridge-rule-pattern.png
```

---

## 🚀 Setup & Deployment

### Prerequisites
- AWS account with admin access
- AWS CLI configured
- Python 3.x

### Step 1 — Enable AWS Config
1. Go to **AWS Config → Get Started**
2. Enable recording for **S3 Bucket** resource type
3. Add these managed rules:
   - `s3-bucket-public-read-prohibited`
   - `s3-bucket-public-read-write-prohibited`
   - `root-account-mfa-enabled`
   - `restricted-ssh`

### Step 2 — Create the Lambda Function
1. Go to **AWS Lambda → Create Function**
2. Name it `auto-remediate-s3`, runtime **Python 3.x**
3. Paste the code from `lambda_function.py`
4. Attach the IAM policy from `iam-policy.json` to the Lambda execution role

### Step 3 — Create the EventBridge Rule
1. Go to **Amazon EventBridge → Rules → Create Rule**
2. Name it `config-auto-remediation`
3. Set event source to **AWS events**
4. Paste the event pattern from `eventbridge-rule.json`
5. Set the target to **AWS Lambda** → select `auto-remediate-s3`

### Step 4 — Enable CloudTrail
1. Go to **AWS CloudTrail → Create Trail**
2. Name it `audit-trail`
3. Store logs in a new S3 bucket

### Step 5 — Test It
1. Create an S3 bucket and **disable** Block Public Access
2. Wait for AWS Config to detect the violation (~2–5 minutes)
3. EventBridge triggers Lambda automatically
4. Verify the bucket now has Block Public Access **enabled**
5. Check CloudWatch Logs for execution details

---

## 🎯 Features

- ✅ Continuous compliance monitoring via AWS Config
- ✅ Event-driven automation with zero manual steps
- ✅ Targeted remediation using event payload (only fixes the flagged bucket)
- ✅ Full audit trail via CloudTrail and CloudWatch Logs

---

## 📸 Screenshots

All screenshots are available in the `screenshots/` folder showing:
- AWS Config dashboard before and after remediation
- Noncompliant S3 bucket detection
- EventBridge rule configuration
- Lambda function setup
- CloudTrail trail configuration
- Final compliant state (0 noncompliant resources)

---

## 🔐 Security Notes

- No credentials or access keys are included in this repository
- The Lambda function uses the **principle of least privilege** — only the permissions required for S3 remediation and CloudWatch logging are granted
- All actions are logged in CloudTrail for full auditability

---

## 👩‍💻 Author

**Shruti Sharma**  
AWS Cloud Security Project — 2026