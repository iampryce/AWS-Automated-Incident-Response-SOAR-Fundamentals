
<p align="center">
  <img src="https://hawk-eye.io/wp-content/uploads/2024/02/SOAR-Features.jpg" width="150">
</p>

<h1 align="center"> AWS Automated Incident Response (SOAR Fundamentals)
</h1>

<p align="center">

</p>

## Project Overview

In modern SOC environments, detecting security incidents is only the first step. Without rapid response, attackers can escalate privileges, maintain persistence, or expand their impact.
This project demonstrates how AWS native services can be used to automatically contain security incidents immediately after detection, following SOC and SOAR best practices.


## 🛠️ AWS Services Used

✔ AWS CloudTrail

✔ Amazon CloudWatch

✔ Amazon EventBridge

✔ AWS Lambda

✔ AWS Identity and Access Management (IAM)

✔ Amazon SNS


## 🗜 Architecture Overview

- Suspicious IAM activity is detected using CloudWatch metrics or GuardDuty findings

- EventBridge captures the detection event

- AWS Lambda executes an automated response action

- SNS notifies the SOC team of the action taken

- All actions are logged for audit and review

## 📚 Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Design](#-architecture-overview)
3. Detection Trigger Selection   
4. SNS Notification Setup
5. IAM Role for Automated Response
6. Lambda Response Function
7. EventBridge Integration
8. Response Testing & Validation
9. Cost Control & Service Shutdown
10. Project Conclusion


## 🧱 Architecture Components & Flow

CloudTrail / GuardDuty

         🔽
        
CloudWatch Metric / Finding

         🔽
        
EventBridge Rule

         🔽
        
Lambda Response Function

         🔽
        
IAM Action + SNS Notification


<img width="1160" height="583" alt="image" src="https://github.com/user-attachments/assets/3d8c5a86-22b3-41e8-92da-58a418ccc200" /> <br/>




### 🔹 1. Detection Source

Security events originate from:

CloudWatch Metric Filters (from CloudTrail logs)

GuardDuty Findings (managed threat detection)

These identify suspicious behavior such as:

Unauthorized IAM activity

Root account usage

Reconnaissance attempts

### 🔹 2. Event Routing (EventBridge)

Amazon EventBridge listens for:

CloudWatch alarms entering ALARM state

GuardDuty findings with severity ≥ Medium

EventBridge acts as the central event router.

### 🔹 3. Automated Response Engine (Lambda)

AWS Lambda executes response actions such as:

Disabling an IAM access key

Attaching a restrictive IAM policy

Logging incident metadata

⚠️ No destructive actions (no deletes) — SOC-safe automation.

### 🔹 4. Notification & Visibility

Amazon SNS sends alerts to:

SOC email

Slack / ticketing (optional extension)

Alerts include:

What happened

Which resource was affected

What action was automatically taken

### 🔹 5. Logging & Audit

All actions are logged to:

CloudWatch Logs

CloudTrail for compliance and audit

