# Automate Infrastructure Compliance with AWS Config

## 📋 Project Overview
Managing and securing a growing inventory of cloud resources manually is inefficient and prone to human error. As a cybersecurity professional and system administrator, I built this project to establish a robust, automated security governance framework using AWS Config.

This project demonstrates how to continuously monitor AWS resources (Amazon EC2 and Amazon S3), detect compliance violations in real-time, and rapidly enforce security policies using automated remediation pipelines.

---

## 🛡️ Why Cloud Compliance Matters (Business Impact)

In a modern cloud environment, security cannot rely on manual, point-in-time audits. I designed this architecture with the following Governance, Risk, and Compliance (GRC) principles in mind:

### Mitigating Misconfigurations
Misconfigured cloud resources (like public S3 buckets) are the number one cause of cloud data breaches. Automated compliance ensures that human error is caught and fixed instantly.

### Regulatory Alignment
Organizations must adhere to strict frameworks (like SOC 2, HIPAA, or PCI-DSS). Enforcing rules like encryption, versioning, and access logging ensures the infrastructure remains audit-ready at all times.

### Continuous Security Posture
Transitioning from "annual security reviews" to Continuous Compliance. The system actively monitors the environment 24/7, providing real-time visibility into the organization's security posture.

### Cost Optimization
Terminating non-compliant, rogue, or unauthorized resources (like public EC2 instances) not only secures the perimeter but also prevents unnecessary cloud spend.

---

## 🎯 Key Objectives

- **Define Security Rules**  
  Implement AWS Managed Rules to enforce strict compliance requirements.

- **Continuous Monitoring**  
  Track infrastructure configurations against internal security guidelines.

- **Detect Violations**  
  Identify non-compliant resources (e.g., EC2 instances with public IPs, S3 buckets lacking versioning).

- **Automate Remediation**  
  Use AWS Systems Manager (SSM) Automation documents to instantly fix or terminate non-compliant resources without manual intervention.

---

## 🛠️ Tools & AWS Services Used

- AWS Config — Configuration tracking and compliance evaluation  
- AWS Systems Manager (SSM) — Automated remediation workflows  
- Amazon EC2 — Compute instances used for testing security rules  
- Amazon S3 — Storage buckets secured with versioning, logging, and public-access blocks  
- AWS IAM — Service-linked roles and least-privilege automation permissions  

---

## 🏗️ Architecture & Environment Setup

The environment consists of Amazon EC2 instances located in a public subnet and several Amazon S3 buckets handling different classifications of data (medium to high security).

### Automated Compliance Workflow

1. A configuration change occurs on an AWS resource.
2. AWS Config evaluates the resource against defined security rules.
3. If the resource is non-compliant, AWS Config triggers a Systems Manager Automation Document.
4. The resource is automatically remediated (e.g., terminated or reconfigured).

---

# 🚀 Step-by-Step Implementation

## Phase 1: Setting up AWS Config & Resource Inventory

To begin, I initialized AWS Config using the **1-click setup**, which creates:

- IAM service-linked role: `AWSServiceRoleForConfig`
- An S3 bucket to store configuration history

I then filtered the **Resource Inventory** to identify the EC2 instances and S3 buckets within the project scope.

📸 Screenshot Opportunity  
Take a screenshot of the **AWS Config "Resource Inventory"** page filtered to show EC2 instances and S3 buckets being tracked.

---

## Phase 2: Defining Security Rules

I configured several AWS Config Rules to enforce strict security policies.

### Compute Security
**ec2-instance-no-public-ip**

Ensures no EC2 instance has a public IP address, preventing external exposure and unauthorized access.

### Storage Access
**s3-bucket-public-read-prohibited**

Ensures no S3 buckets allow public read access, preventing data leaks.

### Data Protection
**s3-bucket-versioning-enabled**

Enforces bucket versioning to protect against accidental overwrites, deletions, or ransomware attacks.

### Audit Logging
**s3-bucket-logging-enabled**

Ensures access logging is enabled on high-security buckets to maintain a strict forensic audit trail.

📸 Screenshot Opportunity  
Take a screenshot of the **AWS Config Rules dashboard** showing the newly created rules.

---

## Phase 3: Detecting Violations

Once the rules were active, AWS Config evaluated the inventory and flagged resources that violated the defined policies.

The AWS Config dashboard provides a clear breakdown of:

- Compliant resources
- Non-compliant resources
- Rules currently violated

This provides instant visibility into the overall security health of the environment.

📸 Screenshot Opportunity  
Take a screenshot of the **AWS Config Dashboard** showing non-compliant resources.

---

## Phase 4: Automating Remediation (Self-Healing Infrastructure)

To eliminate manual remediation and reduce **Mean Time to Remediate (MTTR)**, I linked non-compliant rules to **AWS Systems Manager Automation Documents**.

### EC2 Remediation
Rule: `isolated-compute-rule`

Automation Document: `AWS-TerminateEC2Instance`

If an EC2 instance launches with a public IP, it is automatically terminated.

### S3 Versioning Remediation
Rule: `s3-bucket-versioning-on`

Automation Document: `AWS-ConfigureS3BucketVersioning`

Automatically enables versioning on the bucket.

### S3 Logging Remediation
Rule: `s3-bucket-logging-on`

Automation Document: `AWS-ConfigureS3BucketLogging`

Automatically routes access logs to a designated logging bucket.

### Security Note

All automated actions were executed using a dedicated **AutomationAssumeRole**, strictly following the **Principle of Least Privilege (PoLP)**. This ensures the automation service only has the exact permissions required to remediate violations.

📸 Screenshot Opportunity  
Take a screenshot of the **Remediation Action configuration screen** showing the SSM Automation document setup.

---

## Phase 5: Verification & Auditing

After remediation triggered, I verified the results across the AWS environment.

### Amazon EC2
Confirmed the non-compliant instance with a public IP was automatically terminated.

### Amazon S3
Verified that:

- Versioning was enabled
- Server access logging was enabled

### Systems Manager
Reviewed the **Automation Execution Logs** to confirm successful remediation.

📸 Screenshot Opportunity  
Take a screenshot showing either:

- EC2 instance in **Terminated** state  
or  
- Systems Manager Automation executions showing **Success**

---

# 💡 Conclusion

This project demonstrates the power of **Security by Design**.

By shifting from manual audits to an **automated continuous compliance model**, organizations can:

- Reduce cloud misconfiguration risks
- Maintain audit readiness for compliance frameworks
- Improve incident response times
- Enable security teams to focus on proactive improvements instead of reactive fixes

Automated compliance pipelines like this significantly strengthen an organization's overall cloud security posture.
