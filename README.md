# AWS S3 Labs Technical Documentation

**Author**: Cletus-Nehinlalei-Mangu  
**Date**: August 20, 2025  
**Course**: DevOps Training - Solutions Architect Associate Preparation  

## Introduction

This document serves as a technical guide for the AWS S3 labs I completed as part of my training to prepare for the AWS Solutions Architect Associate certification. The labs focused on three key Amazon S3 features: **Storage Classes & Lifecycle Policies**, **Versioning & MFA Delete**, and **Cross-Region Replication (CRR)**. Each lab provided hands-on experience with S3’s core functionalities, enabling me to understand storage management, data protection, and disaster recovery strategies in a cloud environment. Below, I detail the objectives, setup, processes, challenges, and insights from each lab, along with diagrams and relevant scripts or commands to replicate the work.

This documentation is reflects my experience, including my observations, challenges, and recommendations for applying these concepts in real-world scenarios. It is intended for technical audiences, such as DevOps engineers, cloud architects, or students preparing for AWS certifications.

## Lab Overview

The labs were designed to build practical skills in managing S3 buckets and optimizing storage for cost, performance, and compliance. Below is a summary of each lab’s focus:

1. **Lab 1: Storage Classes & Lifecycle Policies**  
   - I created a bucket and uploaded files to different storage classes (Standard, Standard-IA, Intelligent-Tiering).  
   - I Configured lifecycle policies to transition objects to cheaper storage tiers and expire them after a set period.  
   - And also analyzed cost implications and real-world applications.

2. **Lab 2: Versioning & MFA Delete**  
   - I enabled versioning on a bucket to maintain multiple versions of objects.  
   - And I tested delete markers and object restoration.  
   - Then I documented the security benefits and operational considerations of MFA Delete.

3. **Lab 3: Cross-Region Replication (CRR)**  
   - In this last lab I set up replication between buckets in different AWS regions.  
   - And configured an IAM role for replication and tested object synchronization.  
   - Also I evaluated disaster recovery and cost optimization strategies.

## Prerequisites

Before starting the labs, I ensured the following prerequisites were met:

- **AWS Account**: I used an AWS account with sufficient permissions to create and manage S3 buckets, IAM roles, and related resources.  
- **AWS Console Access**: I accessed the AWS Management Console via `https://console.aws.amazon.com`.  
- **Local Files**: I created the following test files on my computer:  
  - `test1.txt` (for Standard storage)  
  - `test2.txt` (for Standard-IA storage)  
  - `test3.txt` (for Intelligent-Tiering storage)  
  - `version-test.txt` (for versioning tests)  
  - `crr-test-1.txt`, `crr-test-2.txt`, `crr-test-3.txt` (for CRR tests)  

Example content for `test1.txt`:
```
This is test file 1 for Standard storage class.
Created for S3 lab by Cletus-Nehinlalei-Mangu
Date: August 20, 2025
```

- **AWS CLI (Optional)**: Although not used extensively in these labs, I installed the AWS CLI for potential automation tasks and MFA Delete configuration. Command to verify installation:
  ```bash
  aws --version
  ```

- **Documentation Tools**: I used a text editor to document observations and a screenshot tool to capture key steps for submission.

## Lab 1: Storage Classes & Lifecycle Policies

### Objective
The goal was to understand S3 storage classes and automate transitions between them using lifecycle policies to optimize costs.

### Setup and Execution

1. **Bucket Creation**  
   I created a bucket named `cnm-lab1-lifecycle-20250819` in the `us-east-1` region with the following settings:  
   - Object Ownership: ACLs disabled  
   - Block Public Access: Enabled (all checkboxes checked)  
   - Bucket Versioning: Disabled  
   - Default Encryption: SSE-S3  

   AWS CLI command (alternative method):
   ```bash
   aws s3api create-bucket --bucket cnm-lab1-lifecycle-20250819 --region us-east-1
   ```

2. **File Uploads**  
   I uploaded three files with specific storage classes:  
   - `test1.txt` → Standard  
   - `test2.txt` → Standard-IA  
   - `test3.txt` → Intelligent-Tiering  

   CLI command example for uploading `test1.txt`:
   ```bash
   aws s3 cp test1.txt s3://cnm-lab1-lifecycle-20250819/test1.txt --storage-class STANDARD
   ```

3. **Lifecycle Policy Configuration**  
   I created a lifecycle rule named `lab1-lifecycle-rule` to transition objects as follows:  
   - 30 days → Standard-IA  
   - 90 days → Glacier Flexible Retrieval  
   - 365 days → Glacier Deep Archive  
   - 2,555 days → Expire  

   CLI command to create the lifecycle rule:
   ```bash
   aws s3api put-bucket-lifecycle-configuration --bucket cnm-lab1-lifecycle-20250819 --lifecycle-configuration '{
       "Rules": [
           {
               "ID": "lab1-lifecycle-rule",
               "Status": "Enabled",
               "Filter": {},
               "Transitions": [
                   {
                       "Days": 30,
                       "StorageClass": "STANDARD_IA"
                   },
                   {
                       "Days": 90,
                       "StorageClass": "GLACIER"
                   },
                   {
                       "Days": 365,
                       "StorageClass": "DEEP_ARCHIVE"
                   }
               ],
               "Expiration": {
                   "Days": 2555
               }
           }
       ]
   }'
   ```

4. **Verification**  
   I confirmed the rule in the Management tab and manually changed `test1.txt` to Standard-IA to test storage class transitions.

### Diagram
Below is a diagram illustrating the lifecycle transitions:

```
[Standard] --30 days--> [Standard-IA] --90 days--> [Glacier] --365 days--> [Deep Archive] --2555 days--> [Expired]
```

### Challenges
- The Standard-IA 30-day minimum storage warning was initially confusing, as it impacts cost calculations.  
- Navigating to the Management tab for lifecycle rules was not intuitive.  
- Understanding cost trade-offs (e.g., retrieval fees vs. storage savings) required careful analysis.

### Insights
- Lifecycle policies are critical for cost optimization in production environments.  
- Real-world applications include archiving old data (e.g., logs, backups) to Glacier or Deep Archive.  
- Monitoring storage class usage is essential to avoid unexpected retrieval costs.

## Lab 2: Versioning & MFA Delete

### Objective
This lab focused on enabling versioning to maintain object history and understanding MFA Delete for enhanced security.

### Setup and Execution

1. **Bucket Creation**  
   I created a bucket named `cnm-lab2-versioning-20250819` in `us-east-1` with versioning enabled:  
   - Bucket Versioning: Enabled  
   - Other settings: Default (ACLs disabled, public access blocked, SSE-S3 encryption)  

   CLI command:
   ```bash
   aws s3api create-bucket --bucket cnm-lab2-versioning-20250819 --region us-east-1
   aws s3api put-bucket-versioning --bucket cnm-lab2-versioning-20250819 --versioning-configuration Status=Enabled
   ```

2. **Versioning Tests**  
   - Uploaded `version-test.txt` (Version 1).  
   - Modified and re-uploaded the file (Version 2).  
   - Toggled “Show versions” to confirm both versions existed with unique version IDs.  

   CLI command to upload versions:
   ```bash
   aws s3 cp version-test.txt s3://cnm-lab2-versioning-20250819/version-test.txt
   ```

3. **Delete Marker Testing**  
   - Deleted `version-test.txt` (created a delete marker).  
   - Verified the object appeared deleted in normal view but was recoverable.  
   - Restored the object by deleting the delete marker.  

   CLI command to delete and create a delete marker:
   ```bash
   aws s3api delete-object --bucket cnm-lab2-versioning-20250819 --key version-test.txt
   ```

4. **MFA Delete Documentation**  
   I noted that MFA Delete requires root user access and an MFA token for permanent deletions or versioning suspension.

### Diagram
Below is a diagram/flow of versioning behavior:

```
[Version 1] --> [Version 2] --> [Delete Marker] --> [Restore by Deleting Marker]
```

### Challenges
- The “Show versions” toggle was not immediately obvious in the console.  
- Version IDs being system-generated made tracking complex.  
- Storage costs for multiple versions were a concern without lifecycle policies.

### Insights
- Versioning is ideal for protecting against accidental deletions and maintaining audit trails.  
- MFA Delete enhances security but requires careful root account management.  
- Lifecycle policies are necessary to manage version storage costs.

## Lab 3: Cross-Region Replication (CRR)

### Objective
The goal was to implement CRR to replicate objects between buckets in different regions for disaster recovery and compliance.

### Setup and Execution

1. **Bucket Creation**  
   - Source bucket: `cnm-lab3-source-20250819` in `us-east-1` (versioning enabled).  
   - Destination bucket: `cnm-lab3-destination-20250819` in `us-west-2` (versioning enabled).  

   CLI commands:
   ```bash
   aws s3api create-bucket --bucket cnm-lab3-source-20250819 --region us-east-1
   aws s3api create-bucket --bucket cnm-lab3-destination-20250819 --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2
   aws s3api put-bucket-versioning --bucket cnm-lab3-source-20250819 --versioning-configuration Status=Enabled
   aws s3api put-bucket-versioning --bucket cnm-lab3-destination-20250819 --versioning-configuration Status=Enabled
   ```

2. **IAM Role Creation**  
   - Created `lab3-s3-replication-role` with `AmazonS3FullAccess` policy.  
   - Noted the Role ARN for replication configuration.  

   CLI command (simplified, requires JSON policy file):
   ```bash
   aws iam create-role --role-name lab3-s3-replication-role --assume-role-policy-document file://trust-policy.json
   aws iam attach-role-policy --role-name lab3-s3-replication-role --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
   ```

3. **Replication Rule Configuration**  
   - Created a rule named `lab3-crr-rule` to replicate all objects to the destination bucket.  
   - Destination storage class: Standard-IA.  
   - Enabled replication metrics, disabled delete marker replication.  

   CLI command:
   ```bash
   aws s3api put-bucket-replication --bucket cnm-lab3-source-20250819 --replication-configuration '{
       "Role": "arn:aws:iam::ACCOUNT_ID:role/lab3-s3-replication-role",
       "Rules": [
           {
               "ID": "lab3-crr-rule",
               "Status": "Enabled",
               "Prefix": "",
               "Destination": {
                   "Bucket": "arn:aws:s3:::cnm-lab3-destination-20250819",
                   "StorageClass": "STANDARD_IA"
               },
               "DeleteMarkerReplication": { "Status": "Disabled" }
           }
       ]
   }'
   ```

4. **Testing Replication**  
   - Uploaded `crr-test-1.txt`, `crr-test-2.txt`, and `crr-test-3.txt` to the source bucket.  
   - Verified replication to the destination bucket within 2–3 minutes.  
   - Tested delete markers (not replicated) and version updates (replicated correctly).

### Diagram
Below is a diagram of the CRR process:

```
[Source Bucket: us-east-1] --> [Replication Rule] --> [Destination Bucket: us-west-2]
   - Objects: crr-test-*.txt        - Standard-IA storage
   - Versioning enabled             - Versioning enabled
   - Delete markers not replicated
```

### Challenges
- IAM role permissions were overly broad; production requires least-privilege policies.  
- Replication delays (2–4 minutes) must be considered for disaster recovery planning.  
- Storage class conversion to Standard-IA introduced retrieval cost considerations.

### Insights
- CRR is powerful for disaster recovery and compliance but incurs data transfer and storage costs.  
- Monitoring replication metrics is critical for production environments.  
- Selective replication (by prefix or tag) can optimize costs.

## Cleanup

To avoid ongoing charges, I deleted all resources after completing the labs:

1. **Lab 1 Cleanup**  
   - Deleted all objects in `cnm-lab1-lifecycle-20250819`.  
   - Deleted the bucket.  

   CLI command:
   ```bash
   aws s3 rm s3://cnm-lab1-lifecycle-20250819 --recursive
   aws s3api delete-bucket --bucket cnm-lab1-lifecycle-20250819
   ```

2. **Lab 2 Cleanup**  
   - Enabled “Show versions” and deleted all versions and delete markers in `cnm-lab2-versioning-20250819`.  
   - Deleted the bucket.  

   CLI command:
   ```bash
   aws s3api delete-objects --bucket cnm-lab2-versioning-20250819 --delete '{"Objects": [{"Key": "version-test.txt"}]}'
   aws s3api delete-bucket --bucket cnm-lab2-versioning-20250819
   ```

3. **Lab 3 Cleanup**  
   - Emptied both `cnm-lab3-source-20250819` and `cnm-lab3-destination-20250819`.  
   - Deleted both buckets.  
   - Deleted the IAM role `lab3-s3-replication-role`.  

   CLI commands:
   ```bash
   aws s3 rm s3://cnm-lab3-source-20250819 --recursive
   aws s3 rm s3://cnm-lab3-destination-20250819 --recursive
   aws s3api delete-bucket --bucket cnm-lab3-source-20250819
   aws s3api delete-bucket --bucket cnm-lab3-destination-20250819
   aws iam detach-role-policy --role-name lab3-s3-replication-role --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
   aws iam delete-role --role-name lab3-s3-replication-role
   ```

## Key Takeaways

- **Storage Classes & Lifecycle Policies**: Automating transitions to cheaper storage tiers (e.g., Glacier, Deep Archive) is critical for cost optimization, but retrieval fees and minimum storage durations must be considered.  
- **Versioning & MFA Delete**: Versioning protects against accidental deletions and supports compliance, while MFA Delete adds a security layer but requires careful root account management.  
- **Cross-Region Replication**: CRR enhances disaster recovery and data availability but introduces costs and replication delays that must be planned for.  

## Recommendations for Production

- **Cost Management**: Use lifecycle policies to manage versioning and storage class transitions. Monitor costs with AWS Cost Explorer.  
- **Security**: Enable MFA Delete for critical buckets and use least-privilege IAM policies for replication.  
- **Monitoring**: Set up CloudWatch alarms for replication failures and storage growth.  
- **Testing**: Regularly test restoration from Glacier/Deep Archive and CRR failover to ensure disaster recovery readiness.

## Submission Structure

I organized my deliverables in the following folder structure:

```
Cletus-Nehinlalei-Mangu-S3-Labs/
├── Lab1-Screenshots/
│   ├── bucket-creation.png
│   ├── objects-storage-classes.png
│   ├── lifecycle-rule-config.png
│   ├── lifecycle-rule-summary.png
│   └── cleanup-confirmation.png
├── Lab2-Screenshots/
│   ├── versioning-enabled.png
│   ├── multiple-versions.png
│   ├── delete-marker.png
│   ├── object-restoration.png
│   └── cleanup-confirmation.png
├── Lab3-Screenshots/
│   ├── both-buckets-different-regions.png
│   ├── iam-role-created.png
│   ├── replication-rule-config.png
│   ├── replication-results.png
│   └── cleanup-confirmation.png
└── Cletus-Nehinlalei-Mangu-S3-Lab-Report.pdf
```

