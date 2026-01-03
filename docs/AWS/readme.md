# Building an End-to-End Security Monitoring & Alerting Solution in AWS

## Project Overview

This project demonstrates the implementation of a proactive, "defense-in-depth" security framework within an AWS environment. The goal was to move beyond simple logging to create a responsive system that automatically detects, categorizes, and alerts on security threats.

By the end of this project, I established a system that can:

- Log Activity: Record all administrative actions (CloudTrail).

- Alert Instantly: Trigger notifications for high-risk configuration changes (EventBridge).

- Detect Patterns: Identify suspicious behavior, such as brute-force attacks (CloudWatch Alarms).

- Investigate: Perform forensic analysis on log data to identify the source of threats (CloudWatch Logs Insights).

<img width="589" height="259" alt="image" src="https://github.com/user-attachments/assets/b7730515-2255-44f3-8850-6aba0cc6e3b3" />


## Tools & Services Used 

- AWS CloudTrail: The primary audit log for the account, tracking "who" did "what" and "when."

- Amazon CloudWatch: The central monitoring hub used for log storage (Logs) and threshold-based alerting (Alarms).

- Amazon SNS (Simple Notification Service): The notification engine used to dispatch email alerts to administrators.

- Amazon EventBridge: An event-driven service used to catch specific, high-priority actions instantly.

- AWS IAM: Used to simulate real-world users and test authentication failure scenarios.

### Task 1: Building the Foundation (Logging) 

Security begins with visibility. While AWS provides basic history, I needed a permanent, searchable record of every action taken in the account.

Implementation:
I configured an AWS CloudTrail "Trail" to capture all management events across the region. Crucially, I integrated this trail with CloudWatch Logs, ensuring that activity wasn't just stored in a bucket but was available for real-time monitoring.

Analogy: The Security Camera & DVR
Think of CloudTrail as a security camera. By linking it to CloudWatch Logs, I connected that camera to a DVR. Now, all "footage" of account activity is saved in one place where it can be searched and monitored live.

For this lab, a trail (LabCloudTrail) was pre-configured to log to a group (CloudTrailLogGroup), which is what I used for the following tasks.

<img width="619" height="447" alt="image" src="https://github.com/user-attachments/assets/28320c3c-5e01-40d4-afd3-bf65ae4bfe3e" />

1.1 CloudTrail Event History showing a 'CreateStack' event record 

<img width="607" height="356" alt="image" src="https://github.com/user-attachments/assets/78babe35-2fe3-4a81-af35-cd7cc0497559" />

1.2 Create trail' page showing CloudWatch Logs 'Enabled'

<img width="658" height="316" alt="image" src="https://github.com/user-attachments/assets/52570846-136f-49f5-8977-ca010ca7c2eb" />

1.3 'Read' and 'Write' management events selected on the setup page

## Task 2: Establishing the Alerting Channel (SNS)

Monitoring is only effective if the right people are notified. I needed a reliable way to receive alerts outside of the AWS console.

Implementation:
I created an SNS Topic (MySNSTopic) to act as the communication channel. I then created an Email Subscription to that topic. After confirming the subscription via email, I had a verified "alarm bell" ready to ring whenever the security system detected a problem.

<img width="624" height="547" alt="image" src="https://github.com/user-attachments/assets/cf3152a6-016d-4776-8b91-1540f23ed325" />

2.1 'Create topic' page showing 'MySNSTopic' and the Access Policy

<img width="619" height="332" alt="image" src="https://github.com/user-attachments/assets/80ce274b-b143-435a-8e47-7db0355bf74f" />

2.2 'Create subscription' dialog showing the Email protocol and my endpoint

<img width="623" height="247" alt="image" src="https://github.com/user-attachments/assets/f0aaada0-60ed-44c0-a6e5-3b5ff7dd66ec" />

2.3 The 'Subscription confirmed!' success page in my browser

## Task 3: Instant Alerts for Firewall Changes (EventBridge)

Some changes are so high-risk they require an immediate response. Altering a Security Group (firewall) is one of them.

Implementation:
I created an EventBridge Rule specifically looking for the `AuthorizeSecurityGroupIngress` API call.

- The Action (Target): I set the target of this rule to be my MySNSTopic SNS topic. 

- Input Transformation: Since raw logs are difficult to read, I used an Input Transformer to extract specific data (like the Security Group ID and time) and format it into a plain-English email.

- The Test: I intentionally added an unauthorized SSH rule to the server.

- The Result: Within seconds, the system triggered an email alert detailing exactly which firewall rule was changed.

<img width="604" height="301" alt="image" src="https://github.com/user-attachments/assets/5e2ed63b-c669-4613-b8a4-762a213e04ba" />


The EventBridge JSON 'Event pattern' for EC2 security groups

<img width="602" height="479" alt="image" src="https://github.com/user-attachments/assets/a5bb3eec-d610-4410-8350-0bc67565d29c" />

The 'Target input transformer' with my custom Input Path and Template 

<img width="603" height="510" alt="image" src="https://github.com/user-attachments/assets/595166da-fb5b-4ccf-8362-650a94267995" />

The final custom-formatted alert email in my inbox because of the ssh tigger

## Task 4: Detecting Brute-Force Attacks (CloudWatch Alarms)

While Task 3 monitored for a single event, Task 4 focused on patterns. A single failed login is a mistake; five failed logins in a minute is a potential attack.

Implementation:

Metric Filter: I created a filter to scan the logs for "Failed authentication" errors.

The Threshold: I established an Alarm that triggers if the failure count reaches 3 or more within 5 minutes.

The Test: I simulated a brute-force attack by attempting to log in as a test user with an incorrect password multiple times.

The Result: The alarm state turned red ("In Alarm"), and I received a notification of the breach.

Analogy: The Guard at the Keypad
This alarm acts like a security guard watching a keypad. The guard ignores a single typo, but if they see three failed attempts in a row, they hit the main alarm button. This ensures we only alert on suspicious patterns, reducing "alert fatigue."

<img width="567" height="256" alt="image" src="https://github.com/user-attachments/assets/39fe2370-2df1-4405-aa80-91c1b7f83b16" />

The Metric Filter pattern for 'Failed authentication'

<img width="637" height="581" alt="image" src="https://github.com/user-attachments/assets/2af840d3-542d-4f90-bc38-3905d55bcc5f" />

The CloudWatch Alarm 'Conditions' (>= 3 over 5 minutes)

<img width="483" height="467" alt="image" src="https://github.com/user-attachments/assets/62e7b0c1-81d4-427b-b6c8-cb3ce30640ae" />

The red 'authentication information is incorrect' error (The Test)

<img width="632" height="205" alt="image" src="https://github.com/user-attachments/assets/88c7079f-7d5e-4262-9a5c-96ec51084065" />

The CloudWatch Alarm in its red 'In alarm' state

<img width="546" height="576" alt="image" src="https://github.com/user-attachments/assets/e604e944-11ce-4120-857f-483f86ffcbba" />

The final ALARM email in my inbox for 'FailedLogins' because of SNS

## Task 5: Forensic Investigation (Logs Insights)

After an alert is received, security teams need to investigate. I used CloudWatch Logs Insights to perform a forensic audit of the failed login attempts.

Implementation:
I ran a SQL-like query to group the failed logins by their source IP address and the specific IAM user being targeted.

Result: The query generated a structured table showing exactly where the "attack" came from, providing the data needed to block the malicious IP address or secure the targeted user.

<img width="654" height="505" alt="image" src="https://github.com/user-attachments/assets/2b6fe9e4-9b5c-4cb4-ae9b-01f01eb32de0" />

Logs Insights page showing the query and the table of failed login results

## Conclusion & Key Learnings

This project successfully demonstrates a multi-layered security posture:

Real-time Detection: Using EventBridge for instant "Tripwire" alerts.

Behavioral Analysis: Using CloudWatch Alarms to catch brute-force patterns.

Forensic Capability: Using Logs Insights for post-incident investigation.

**Key Takeaway:** The difference between a secure environment and a vulnerable one is automation. By chaining these services together, I reduced the "Time to Detect" from hours of manual log checking to a matter of seconds, providing a robust net that protects the cloud infrastructure 24/7.
