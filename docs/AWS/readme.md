# Building an End-to-End Security Monitoring & Alerting Solution in AWS

The goal was not just to create a single alert but to demonstrate a comprehensive, defense-in-depth strategy. I configured a system to: 

1. Log everything (CloudTrail)
2. Alert instantly on high-risk, specific events (EventBridge)
3. Alert on patterns of suspicious behavior (CloudWatch Alarms)
4. Provide a way to forensically query all logs (CloudWatch Logs Insights) 

This project showcases how to chain together multiple AWS services to create a robust and automated security response system.

## Tools & Services Used 

- AWS CloudTrail: The "audit log" for the entire AWS account. I used it as the foundational data source for what API calls were made, who made them, and when.
- Amazon CloudWatch: The central hub for monitoring. I used two of its key features:
     - CloudWatch Logs: The "DVR" that stores, views, and monitors all the log files from CloudTrail.
     - CloudWatch Alarms & Metrics: The "state-based" trigger that fires when a certain threshold (like 3 failed logins) is breached.
- Amazon SNS (Simple Notification Service): The "alarm bell" or "pager." This is the notification service I used to send out email alerts.
- Amazon EventBridge: The "event-driven" trigger. This is the "brain" that filters for very specific events (like a security group change) and fires an alert instantly.
- AWS IAM (Identity and Access Management): Used to create a test user and generate the failed login events.

### Task 1: Setting the Foundation (CloudTrail + CloudWatch Logs) 

Before I could monitor or alert on anything, I had to start collecting the data. By default, CloudTrail shows 90 days of limited history. I needed a permanent, queryable log of all API activity. 

What I did: I walked through the (simulated) process of creating a new CloudTrail trail. The most critical step was configuring the trail to send all log events to a new CloudWatch Logs Group. 

Analogy: Installing the Security Cameras Think of CloudTrail as the security camera pointed at every door and window in your AWS account. By default, it just remembers what it saw for a little while. 

Creating a trail and linking it to CloudWatch Logs is like hooking up all those cameras to a central Digital Video Recorder (DVR). Now, all footage is saved permanently, in one place, where I can easily review and analyze it. 

For this lab, a trail (LabCloudTrail) was pre-configured to log to a group (CloudTrailLogGroup), which is what I used for the following tasks.


Screenshot Placeholders: 

[Screenshot Placeholder: Task 1 - CloudTrail Event History showing a 'CreateStack' event record] 

[Screenshot Placeholder: Task 1 - 'Create trail' page showing CloudWatch Logs 'Enabled' and IAM role selected] 

[Screenshot Placeholder: Task 1 - 'Choose log events' page showing 'Read' and 'Write' management events selected] 





