# Project: Building a "Self-Healing" Infrastructure with AWS Config & Lambda

## Project Overview

In modern cloud environments, security alerts are only effective if they are acted upon quickly. This project moves beyond simple "detection and notification" to achieve **Automated Remediation**. I built a self-healing system that monitors EC2 Security Groups for unauthorized changes and uses a Lambda function to automatically revert those changes within minutes—effectively removing the "human in the loop" for routine security enforcement.

By implementing this architecture, I demonstrated a **Continuous Compliance** model. Rather than relying on periodic audits, the system is designed to maintain a "Desired State" 24/7. If a configuration drifts from the pre-defined security baseline, the automation identifies the discrepancy and performs the corrective action immediately, ensuring the infrastructure remains secure even when administrative errors occur.

---

## The Strategy: Compliance as Code

I implemented a **Continuous Compliance** model where:

- **AWS Config** acts as the continuous monitor (the "Guard").
- **AWS Lambda** acts as the automated response (the "Remediator").
- **CloudWatch Logs** provides the forensic audit trail of every automated fix.

<img width="612" height="325" alt="image" src="https://github.com/user-attachments/assets/c6fb08f3-758f-42be-be1c-6915858a9656" />

---

## Tools & Services Used

- **AWS Config**: Used to inventory resources and evaluate their configuration against security policies.
- **AWS Lambda**: A serverless function containing the Python logic to revoke unauthorized firewall rules.
- **IAM Roles**: Configured with "Least Privilege" to allow Config to monitor and Lambda to modify security groups.
- **Amazon VPC (Security Groups)**: The target resource used to simulate and test security violations.
- **CloudWatch Logs**: The centralized logging service used to verify that the automated remediation took place.

---

## Technical Implementation

### 1. Identity & Access Management (IAM)

Automated systems require precise permissions. I configured two specific roles:

- **AwsConfigRole**: Granted the "Guard" permission to read resource configurations across the account.
- **AwsConfigLambdaSGRole**: Granted the "Remediator" the power to `RevokeSecurityGroupIngress`. This role also includes `config:PutEvaluations`, allowing the function to report its success back to the AWS Config dashboard.

<img width="688" height="248" alt="image" src="https://github.com/user-attachments/assets/5f37177f-e66e-4302-af94-9e4b3281af6c" />

`Task 1 - The 'AwsConfigRole' permissions tab, showing both the 'S3Access' and 'AWS_ConfigRole' policies attached`

<img width="597" height="323" alt="image" src="https://github.com/user-attachments/assets/d10ac206-1355-4e27-8fc9-f207492838a8" />

`Task 1 - IAM Policy showing 'ec2:Revoke' and 'config:PutEvaluations' permissions`

---

### 2. Implementing Continuous Monitoring (AWS Config)

I initialized AWS Config to specifically target AWS EC2 `SecurityGroup` resources. By choosing **Continuous Recording**, the system ensures that any change made to a firewall rule is detected the moment it happens, rather than waiting for a daily scan.

<img width="642" height="329" alt="image" src="https://github.com/user-attachments/assets/3544af59-8b87-4f26-b875-d9f62644e43c" />

`Task 2 - Config Settings showing 'Specific resource types' set to SecurityGroups`

---

### 3. Simulating a Security Violation

To test the system, I acted as a "careless developer" and modified a Security Group (`LabSG1`). I added unauthorized inbound rules for:

- **SMTPS (Port 465)**
- **IMAPS (Port 993)**

This exposed sensitive email ports to the entire internet.

>Note:The last two ports are for email. They are almost never supposed to be open to the entire internet and are a perfect example of a "non-compliant" configuration

<img width="690" height="336" alt="image" src="https://github.com/user-attachments/assets/1ec3ff81-1ab7-443d-816d-ac902018c378" />

`Task 3 - Security Group rules showing unauthorized SMTPS/IMAPS ports added`

---

### 4. Deploying the "Self-Healing" Logic

I linked AWS Config to a custom Lambda Function. This function contains a "blueprint" (a list of allowed ports such as HTTP/80 and HTTPS/443).

#### The Blueprint:
I inspected the Lambda code to see the REQUIRED_PERMISSIONS variable—the literal "rulebook" the system uses.

#### The Trigger:
As soon as Config detected my manual changes, it triggered the Lambda function.

#### The Action:
The code compared the new rules against the allowed blueprint, identified the email ports as unauthorized, and instantly revoked them.

<img width="640" height="726" alt="image" src="https://github.com/user-attachments/assets/0935fce0-ca7a-4bf5-a974-6031352bf3ba" />

`Task 5 - The Lambda function's code, highlighting the 'REQUIRED_PERMISSIONS' array`

<img width="825" height="290" alt="image" src="https://github.com/user-attachments/assets/06d1d5d5-977a-4a75-b536-bbc0e5ca18ee" />

`Task 4 - AWS Config Rule configuration pointing to the Lambda ARN`

<img width="877" height="653" alt="image" src="https://github.com/user-attachments/assets/0e9b71c9-95c3-4814-b4d8-edd93c83adc3" />

`Task 4 - Dashboard showing 'LabSG1' returning to a 'Compliant' status`

---

### 5. Validation and Forensic Auditing

To confirm the system worked as intended, I verified the results in two places:

#### Infrastructure Check:
Returning to the EC2 Console, I confirmed the SMTPS and IMAPS rules were deleted automatically.

#### Audit Trail:
I inspected CloudWatch Logs, where the Lambda function recorded its exact actions:

- `"Revoking for port 465"`
- `"Revoking for port 993"`

<img width="887" height="544" alt="image" src="https://github.com/user-attachments/assets/456227c7-c419-4459-b9c9-1f68ad16b3a7" />

`Task 5 - Security Group rules showing only HTTP/HTTPS remain`

<img width="849" height="316" alt="image" src="https://github.com/user-attachments/assets/aff50ac3-3831-46a8-a536-79bbe013d0c8" />

`Task 6 - CloudWatch Logs showing the 'revoking for' event messages`

---

## Conclusion & Key Learnings

This project demonstrates the power of **Automated Remediation**. By shifting from a human-dependent security model to an automated one, the **Mean Time to Remediate (MTTR)** dropped from hours (waiting for an admin to see an email) to seconds.

### Key Takeaways:

- **Detection vs. Prevention**: While blocking changes at the source is ideal, automated remediation provides a flexible "safety net" that allows for rapid fixes without breaking developer workflows.
- **State Management**: AWS Config is essential for maintaining a "desired state" of infrastructure.
- **The Audit Trail**: Automated actions must always be logged. The CloudWatch logs produced in this project are critical for compliance officers to understand what was fixed and why.

---

This "Self-Healing" architecture is a core pillar of modern DevSecOps, ensuring that security policies are enforced 24/7 without manual intervention.

