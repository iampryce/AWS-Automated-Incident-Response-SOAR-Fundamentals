
<p align="center">
  <img src="https://hawk-eye.io/wp-content/uploads/2024/02/SOAR-Features.jpg" width="150">
</p>

<h1 align="center"> AWS Automated Incident Response 
</h1>

<p align="center">

</p>

## Project Overview
This project demonstrates how AWS native services can be used to automatically contain security incidents immediately after detection, following SOC and SOAR best practices.


## 🛠️ AWS Services Used

✔ AWS CloudTrail

✔ Amazon CloudWatch

✔ Amazon EventBridge

✔ Amazon GuardDuty

✔ AWS Lambda

✔ AWS Identity and Access Management (IAM)

✔ Amazon SNS




## 🗜 Architecture Overview

1. Suspicious IAM activity is detected using CloudWatch metrics or GuardDuty findings

2. EventBridge captures the detection event

3.  AWS Lambda executes an automated response action

4.  SNS notifies the SOC team of the action taken

5. All actions are logged for audit and review

<img width="3796" height="1960" alt="Main drawio" src="https://github.com/user-attachments/assets/9ae8805c-c1b5-410c-a565-65d4b03bbfb9" />




## 📚 Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Design](#-architecture-overview)
3. [Detection Trigger Selection](#-1a-detection-source--prep)
4. [Lambda Response Function](#-3-automated-response-engine-lambda)
5.  [EventBridge Integration](#-2-event-routing-eventbridge)
6.  [ Response Testing & Verification](#-solution-source-chatgpt)
7. [Project Conclusion](#-conclusion)


## 🗜 1A. Detection Source & Prep

🔷 _step 1_

### Created a S3 Bucket to store CloudTrail Logs

🧰  S3 Bucket Configurations

✔ Block ALL public access

✔ Leave defaults enabled

✔ Enable versioning

<img width="1343" height="622" alt="image" src="https://github.com/user-attachments/assets/2db7a53d-051b-447f-a849-439446a2f12d" />

----

🔷 _step 2_

#### Created CloudTrail 

📌 I created a multi-region CloudTrail, configured to capture all management API activity across the AWS account or outside.

📌 Logs are stored in the Amazon S3 bucket for audit purposes and streamed to CloudWatch Logs for real-time security detection.

### 🧰  CloudTrail Configurations

✔ Enable: Apply trail to all regions

✔  Event type : MAnagement event, Captures management operations performed on AWS resources. (IAM changes, EC2 actions, Security group updates, Root activities)

✔ Enable CloudWatch Logs

✔ Enable  Log File Integrity Validation

✔ Enabled CloudWatch 

<img width="1271" height="598" alt="image" src="https://github.com/user-attachments/assets/c70daca4-2dfc-483f-b061-35a7082f79e4" />

<img width="1344" height="560" alt="image" src="https://github.com/user-attachments/assets/004bb1ec-ff4a-4897-8c96-8765b8363ebb" />


## ✅ Verification 

✔ Logging: ON

✔ S3 bucket begins receiving logs : Trail log location
security-cloudtrail-logs01/AWSLogs/o-z96jg5oz46/766593778503 

✔ CloudWatch log group exists


<img width="1334" height="554" alt="image" src="https://github.com/user-attachments/assets/9815ea44-5560-430e-8de1-cdc36de6f65f" /> <br/>

✔ CloudWatch log strems which means cloudTrail is streaming to cloudWatch

<br>

<img width="1288" height="616" alt="image" src="https://github.com/user-attachments/assets/34e34cc4-735d-415f-80e3-65b37fac0c5f" />


✔ CloudTrail logging was validated by generating IAM activity and confirming log delivery to the S3 Bucket. <br/>


<img width="1346" height="620" alt="image" src="https://github.com/user-attachments/assets/74360680-5a54-4fb6-83ae-04073841cd8f" />



## 1B Detection Rule: Unauthorized API Calls 

### Flow       🔽

Security events originate from:

- CloudWatch Metric Filters (from CloudTrail logs)

- GuardDuty Findings (managed threat detection)

### 📌 Purpose: identify suspicious behavior such as:

- Unauthorized IAM activity

- Root account usage

- Reconnaissance attempts 
  

## 🔍 Unauthorized IAM Activity

🔷 _step 1_

### Created a CloudWatch Metric Filter to capture all unauthorised IAM activities

<img width="1137" height="599" alt="image" src="https://github.com/user-attachments/assets/446a2bf7-6dea-4b02-91e1-5f3d72b67a3d" />

### I Created an Alarm for the filter i made and also Configured SNS Alert (This sends notification to my email)

✔ _SNS Configuration_ 

<img width="1347" height="562" alt="image" src="https://github.com/user-attachments/assets/e7a2377c-5e98-4b59-8c28-133fde340d4c" />

✔ _Alarm Created_

<img width="1297" height="531" alt="image" src="https://github.com/user-attachments/assets/af4fc02d-1c6b-4a79-833f-62e5aaff67e9" />


## ✅ Verification

📌 Note: In other to trigger alarm, i created a low priviledge user and performed some IAM activity. (This is to confirm that everything works)

- ✔ SNS Eamil received

<img width="626" height="1280" alt="image" src="https://github.com/user-attachments/assets/b4255a14-949a-444f-a00a-a78ca6a087de" />

- ✔ CloudWatch Alarm triggered
  
<img width="865" height="566" alt="image" src="https://github.com/user-attachments/assets/6c6c8a0d-3cc0-48df-81fa-d65e96d83dcd" />

- ✔ CloudWatch Metrics confrimed (Everything wroks fine)
  
<img width="1276" height="565" alt="image" src="https://github.com/user-attachments/assets/643b011d-de76-4285-b25a-c393dab47601" />


## 1C I Integrated GuardDuty to automatically identify malicious activity using AWS threat intelligence and ML

📌 Note: Amazon GuardDuty was enabled to provide managed threat detection using AWS machine learning and threat intelligence. GuardDuty complements custom CloudTrail-based detections by identifying suspicious behavior without manual rule creation.

✔ GuardDuty enabled

<img width="1332" height="586" alt="image" src="https://github.com/user-attachments/assets/499d0a2a-3ce7-417b-b237-9d2103146159" />



## 🗜 2. Event Routing (EventBridge)

### 📌 Purpose: Automatically route GuardDuty high-severity findings to the response engine (Lambda) for real-time action.

### Flow       🔽

Amazon EventBridge listens for:

- CloudWatch alarms entering ALARM state

- GuardDuty findings with severity ≥ Medium

  EventBridge acts as the central event router.


###  (AWS Lambda function) I created Lambda function to be the targert in the event bridge configuration

✔ Event bridge created and a Lambda Fuction set as the target with a role assigned

📌 Notes: Event pattern shows souce as aws.guardduty which means it only capture GuardDuty findings.  >= 5 means medium to high severity (severity ranges from 0 - 8)

<img width="1313" height="565" alt="image" src="https://github.com/user-attachments/assets/229a8ab0-2c28-4ea2-97cb-fc819d18d319"/>


## ✅ Verification

📌 Note: I generated GuardDuty sample findings to validate end-to-end event routing and Lambda invocation.

✔  Verified GuardDuty Findings

<img width="1322" height="570" alt="image" src="https://github.com/user-attachments/assets/8fd26f73-daf2-4f06-87fd-f206877d7f1d" />


✔ Verified EventBridge Rule Matches
  
<img width="1316" height="540" alt="image" src="https://github.com/user-attachments/assets/0af262ca-401b-487c-b46e-7f20123104c9" />

📌 #### Troubleshooting 1

❌ During validation, I used Lambda manual invocation to confirm logging functionality. Earlier EventBridge-triggered invocations executed a previous Lambda version before code deployment, resulting in system logs only. Redeploying the function resolved the issue.

<img width="1279" height="549" alt="image" src="https://github.com/user-attachments/assets/734baa4e-aff1-4adb-9c53-0f228a105a8a" />


✔ Verified Lambda Invocation works (Everything works)

<img width="1341" height="615" alt="image" src="https://github.com/user-attachments/assets/7247ae5f-747d-4364-a1ba-078dc52a4bfd" />


## 🗜 3. Automated Response Engine (Lambda)

📌 #### Troubleshooting 2

i wanted the log to include the username of the IAM User who triggered the action, before Lambda restrict the user

❌ Guard duty did not display IAM Username

🚨 Reasons

Most GuardDuty findings DO NOT include an IAM username

🔁 Found out i have to use CloudTrail-based detection for IAM identity Not GuardDuty.



### Flow 🔽

Security events originate from:

CloudTrail logs

AccessDenied API calls

 🔽  Detected by

CloudWatch Metric Filter

 🔽  Alarm triggers

CloudWatch Alarm

 🔽 Routed via

Amazon EventBridge

 🔽  Trigger

AWS Lambda (Automated Response Engine)

 🔽 Action

Disables USER access key 

Logs incident data


🔷 _step 1_

Created a Restrictive IAM Policy

📌 Purpose: Limits IAM user activity.

<img width="1321" height="575" alt="image" src="https://github.com/user-attachments/assets/9225f996-7f1d-4ba3-8b26-40f121f4ad44" />


🔷 _step 2_

Updated Lambda Execution Role and attached IAMFullAccess. This allows Lambda function to detect unauthorized IAM activity and automatically attach a restrictive policy to contain the incident.

📌 Note: IAMFullAcess allows Lambda to identify an IAM user and attach a policy to that user.

<img width="1326" height="636" alt="image" src="https://github.com/user-attachments/assets/59c59553-c1f5-42c6-839a-ee2374951f56" />

🔷 _step 3_

Updated the EventBridge rule to set cloudwatch as the source, this changes the setup from the initial source which was GuardDuty.

Updated the event filter 

<img width="1347" height="626" alt="image" src="https://github.com/user-attachments/assets/dc983cc2-cc3c-4af2-80d8-0798cb8ed177" />

🔷 _step 4_

Updated Lambda Code for containment

_📌 Note: This code was generated with AI_  


<img width="1326" height="641" alt="image" src="https://github.com/user-attachments/assets/59fce0d4-f323-4e71-bb7a-63ee0531ae52" />

✅ Test

Outcome : Log working but no username

<img width="1337" height="639" alt="image" src="https://github.com/user-attachments/assets/79ca53fa-5eb0-49d7-8ae6-8c1e4b0e76d9" />

📌 #### Troubleshooting 3

❌ Cloudwatch log  did not also display IAM Username (it did not display the name of user who performed the action)

So the problem persist.

## ✅ Solution (source ChatGpt)

📌 Note: To dynamically identify the IAM user, you must send CloudTrail events DIRECTLY to EventBridge Not via alarms. 

CloudTrail Records who did what in AWS (API calls, user/role, time, source IP, etc.)

Captures the user identity (userName, arn, role)

CloudWatch Logs

Receives raw log messages from services like Lambda, VPC Flow Logs, or CloudTrail if you forward them

Also i had 2 EventBridge rule enable  for the same lambda so it's clashing



 





Correct Solutions Flow 🔽

Security events originate from:

CloudTrail API events (AccessDenied)

 🔽 Routed directly by

Amazon EventBridge (CloudTrail event source)

 🔽 Trigger

AWS Lambda

 🔽 Action

Extract IAM user 

🔽

Lambda disables the user’s access key

 🔽
 
SNS notifies SOC

 🔽
 
All actions logged for audit

🔷 _step 1_

Created an EventBridge Rule (CloudTrail API Events)

In this case EventBridge will route CloudTrail API event to the targert Lambda while Lambda function get triggered.

<img width="1319" height="597" alt="image" src="https://github.com/user-attachments/assets/390f6a1c-a77a-423f-b4d8-06fa0eea8b85" />


 <br/>


🔷 _step 2_

Updated Lambda function code to disable user access key

<img width="1344" height="626" alt="image" src="https://github.com/user-attachments/assets/85d1e270-07fb-4999-a7c2-8f557f5ae6e8" />





🔷 _step 3_


## ✅ Verification

### ✅ Test 1: Unauthorized IAM Action ( Extracted IAM user )

<img width="1302" height="639" alt="image" src="https://github.com/user-attachments/assets/d80954c1-ba17-484b-9a02-858e9649eb60" />


<br/>


### ✅ Test 2: Unauthorized IAM Action ( Extracted IAM user )

<img width="1366" height="556" alt="image" src="https://github.com/user-attachments/assets/73f96ca0-1e67-4ab9-9300-ac701215fc11" />


<img width="1318" height="599" alt="image" src="https://github.com/user-attachments/assets/03ca894e-75a0-4cbe-a765-66fbf4f0a06b" />

## ✅  Final verification

CloudWatch logs confirms the USER and includes name. 

<img width="1338" height="603" alt="image" src="https://github.com/user-attachments/assets/2833d6c5-7f33-47c5-af95-ad505e9ecf9b" />




## 🗜 4. Notification & Visibility

Amazon SNS sends alerts to:

Email

Alerts include:

What happened

Which resource was affected

### ✅


![photo_2026-01-14_14-09-11](https://github.com/user-attachments/assets/64a744cc-0e8c-4ce5-9791-509008515e82)


## 🔥 Conclusion

“CloudWatch logs now include username but it said no user access key found”

This tells us 3 critical things:

EventBridge → Lambda	✅ Working

Lambda parsing CloudTrail	✅ Correct

Username detection	✅ SUCCESS

Access key detection	⚠️ Conditional (not always present) 


