<div align="center">

# 🔐 AWS Cloud Security Lab

### Hands-on cloud security engineering — VPC architecture, IAM enforcement, threat detection, and misconfiguration remediation on AWS.

![AWS](https://img.shields.io/badge/Amazon_AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Security](https://img.shields.io/badge/Cloud_Security-232F3E?style=for-the-badge&logo=amazonaws&logoColor=FF9900)
![IAM](https://img.shields.io/badge/IAM-Least_Privilege-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

</div>

---

## 📋 Overview

This lab demonstrates real-world cloud security engineering skills across AWS — from designing secure VPC architecture to enforcing least-privilege IAM policies, enabling audit logging with CloudTrail, and detecting and remediating critical misconfigurations.

> **Every finding in this lab reflects real attack vectors found in production AWS environments.**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                    AWS VPC                          │
│                  10.0.0.0/16                        │
│                                                     │
│   ┌──────────────────┐   ┌──────────────────┐      │
│   │  Public Subnet   │   │  Private Subnet  │      │
│   │  10.0.1.0/24     │   │  10.0.2.0/24     │      │
│   │                  │   │                  │      │
│   │  ┌────────────┐  │   │   (isolated)     │      │
│   │  │  EC2       │  │   │   no IGW route   │      │
│   │  │  + IAM Role│  │   │                  │      │
│   │  └────────────┘  │   └──────────────────┘      │
│   └──────────────────┘                              │
│           │                                         │
│   ┌───────────────┐                                 │
│   │ Route Table   │                                 │
│   │ 0.0.0.0/0 →   │                                 │
│   │ IGW           │                                 │
│   └───────────────┘                                 │
└─────────────────────────────────────────────────────┘
           │
    ┌──────────────┐
    │   Internet   │
    │   Gateway    │
    └──────────────┘
           │
        Internet
```

---

## 🛠️ Services Used

| Service | Purpose |
|---------|---------|
| **Amazon VPC** | Custom network architecture with public/private segmentation |
| **Amazon EC2** | Amazon Linux 2023 instance in public subnet |
| **Amazon S3** | Storage bucket with versioning and block public access |
| **AWS IAM** | Least-privilege role and policy attached to EC2 |
| **AWS CloudTrail** | Multi-region API audit logging |
| **AWS Security Groups** | Stateful firewall restricting inbound access |

---

## 🔬 Lab Steps

### 1️⃣ VPC Architecture

Built a custom VPC with isolated public and private subnets. Public subnet routes to the internet via IGW. Private subnet has no internet route — fully isolated by design.

**Security controls applied:**
- ✅ Network segmentation via separate subnets
- ✅ Private subnet with no IGW route
- ✅ Dedicated route table per subnet

---

### 2️⃣ Security Group Configuration

Deployed a security group restricting SSH (port 22) to authorized IPs only.

> ⚠️ **Finding:** SSH open to `0.0.0.0/0` is one of the most common critical misconfigurations in AWS environments — exposing instances to brute force attacks from the entire internet.

---

### 3️⃣ IAM Least-Privilege Enforcement

Created a custom IAM role with a scoped policy — EC2 can only read from one specific S3 bucket. Nothing else.

<details>
<summary>📄 View IAM Policy</summary>

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::kency-security-lab-bucket",
        "arn:aws:s3:::kency-security-lab-bucket/*"
      ]
    }
  ]
}
```

</details>

**Validation from inside EC2:**

```bash
# ✅ ALLOWED — list bucket
aws s3 ls s3://kency-security-lab-bucket
> 2026-05-03 22:00:52    43 test file text.txt

# ✅ ALLOWED — download file
aws s3 cp s3://kency-security-lab-bucket/test file text.txt .
> download: s3://kency-security-lab-bucket/test file text.txt

# ❌ DENIED — create bucket
aws s3 mb s3://kency-unauthorized-bucket-test
> AccessDenied: not authorized to perform: s3:CreateBucket

# ❌ DENIED — delete bucket
aws s3 rb s3://kency-security-lab-bucket
> AccessDenied: not authorized to perform: s3:DeleteBucket
```

---

### 4️⃣ CloudTrail Audit Logging

Enabled a multi-region CloudTrail trail capturing all management API calls. Every IAM change, S3 operation, and EC2 action is logged and stored in a dedicated S3 bucket.

**Why this matters:** Without CloudTrail, there is no audit trail. If credentials are compromised, you have no visibility into what an attacker did or accessed.

---

### 5️⃣ Misconfiguration Detection & Remediation

Simulated a critical IAM misconfiguration and documented the full detection-to-remediation cycle.

<details>
<summary>🚨 View Misconfigured Policy (Finding)</summary>

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

</details>

<details>
<summary>✅ View Remediated Policy (Fixed)</summary>

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::kency-security-lab-bucket",
        "arn:aws:s3:::kency-security-lab-bucket/*"
      ]
    }
  ]
}
```

</details>

---

## 🚨 Findings Summary

| # | Finding | Severity | Status |
|---|---------|----------|--------|
| 1 | Overly permissive IAM policy — `s3:*` on `Resource: *` | 🔴 High | ✅ Remediated |
| 2 | SSH open to `0.0.0.0/0` | 🟡 Medium | ✅ Documented |

---

## 💡 Security Concepts Demonstrated

```
Least Privilege      → IAM policy scoped to specific actions + resource ARN
Network Segmentation → Public/private subnets with separate route tables
Defense in Depth     → SG + IAM + CloudTrail as layered controls
Audit Logging        → CloudTrail capturing all API calls for forensics
Misconfiguration     → Detecting and remediating s3:* on Resource: *
Access Validation    → Proving controls work via live AWS CLI testing
```

---

## 📸 Screenshots

### 🏗️ VPC & Subnet Architecture
<img width="1632" height="356" alt="image" src="https://github.com/user-attachments/assets/718e9376-017c-4518-b738-ba5ef509b63a" />



### 🔒 IAM Validation — Access Denied Output
<img width="2854" height="340" alt="image" src="https://github.com/user-attachments/assets/3f174b1a-6af8-426d-bfc8-ab34b7578530" />


### 🚨 Misconfigured Policy — Before
<img width="1213" alt="Overly permissive IAM policy s3:* on Resource:*" src="https://github.com/user-attachments/assets/47470d22-8276-453b-a8b3-ec84df4b1148" />

### ✅ Remediated Policy — After
<img width="1211" alt="Least-privilege IAM policy scoped to specific bucket" src="https://github.com/user-attachments/assets/f31de4b6-04e2-49dc-aa4f-88cce25b296c" />

### 📋 CloudTrail Trail Active
<img width="2450" height="518" alt="image" src="https://github.com/user-attachments/assets/e5fb47b3-7388-404d-ab87-0de2571d1633" />

---

## 🗺️ What's Next

- [ ] Add GuardDuty for automated threat detection
- [ ] Add AWS Config rules for continuous compliance
- [ ] Add CloudWatch alarms for unusual API activity
- [ ] Add VPC Flow Logs for network traffic visibility
- [ ] Rebuild entire lab using Terraform (IaC)

---

<div align="center">

**Built by [Kency Francois](https://linkedin.com/in/kency-francois)**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/kency-francois)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/kfran127)

</div>
