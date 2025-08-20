# AWS S3 Lab 3 Report: Cross-Region Replication

**Student Name:** Cletus-Nehinlalei-Mangu  
**Course:** DevOps Training - Solutions Architect Associate Preparation  

---

## Executive Summary
This lab focused on implementing cross-region replication (CRR) between S3 buckets in different AWS regions.  
The exercise demonstrated **disaster recovery capabilities, geographic data distribution, and automated data synchronization** across regions for enhanced availability and compliance.

---

## Lab Objectives Completed
I Successfully achieved all primary objectives:

- ✅ Created source bucket `cnm-lab3-source-20250819` in `us-east-1`  
- ✅ Created destination bucket `cnm-lab3-destination-20250819` in `us-west-2`  
- ✅ Configured IAM role `lab3-s3-replication-role` with appropriate permissions  
- ✅ Implemented cross-region replication rule with storage class optimization  
- ✅ Tested replication behavior including new uploads, deletions, and modifications  

---

## Implementation Process

### Multi-Region Bucket Setup
- Both my buckets were created successfully with **versioning enabled** (required for CRR).  
- Geographic separation between `us-east-1` (Virginia) and `us-west-2` (Oregon) provided realistic disaster recovery testing.

### IAM Role Configuration
- I Created dedicated IAM role with **AmazonS3FullAccess** policy.  
- While broader than production requirements, this allowed replication setup.  
- The Role ARN was captured for replication rule configuration.

### Replication Rule Implementation
Configured with:
- All objects in source bucket scope  
- Destination storage class: **Standard-IA** (cost optimization)  
- Replication metrics enabled for monitoring  
- Delete marker replication **disabled** to prevent cascading deletions  

---

## Functional Testing Results

### Object Replication
- Test objects (`crr-test-1.txt`, `crr-test-2.txt`) replicated successfully within **2–3 minutes**.  
- Verified:
  - Content preserved  
  - Storage class converted to Standard-IA  
  - Different version IDs across regions  
  - Metadata & timestamps maintained  

### Delete Behavior Analysis
- Deleting `crr-test-1.txt` in source:
  - Delete marker created in source  
  - Object remained intact in destination  
  - Delete marker not replicated → protection against accidental deletions  

### Version Replication
- Modified `crr-test-2.txt` → new version replicated correctly.  
- Both old and new versions existed in destination.

### Additional Object Testing
- Upload of `crr-test-3.txt` confirmed ongoing replication worked consistently.

---

## Challenges and Solutions

- **IAM Permission Complexity**  
  - Used AmazonS3FullAccess due to uncertainty.  
  - Production should use least-privilege custom policy:  
    - Source bucket read  
    - Destination bucket write  
    - Version/metadata replication  
    - Storage class conversion  

- **Replication Timing Variability**  
  - Observed delays (1–4 minutes).  
  - Must be factored into disaster recovery planning.  

- **Storage Class Conversion**  
  - Replication automatically converted Standard → Standard-IA.  
  - Requires awareness of retrieval costs.  

- **Cross-Region Dependencies**  
  - Replication depends on AWS inter-region networking.  
  - Must be considered in HA design.  

---

## Real-World Applications

### Disaster Recovery
- Automatic backup across regions  
- Reduced **RPO** via near real-time replication  
- Compliance with regulatory requirements  

### Global Content Distribution
- Lower latency for end users worldwide  
- Load distribution & content localization  

### Regulatory Compliance
- Meets **data sovereignty laws**  
- Maintains audit trails across jurisdictions  

### Multi-Region Applications
- Consistent app data across regions  
- Failover readiness  
- Useful for read replicas & test environments  

---

## Cost Analysis

### Replication Costs
- Replication requests: **$0.0004 / 1,000 PUTs**  
- Inter-region transfer: **$0.02/GB**  
- Destination storage (Standard-IA): **$0.0125/GB**  
- Monitoring (CloudWatch metrics/alarms)  

### Cost Optimization Strategies
- Standard-IA conversion (≈46% storage savings)  
- Selective replication by prefix  
- Lifecycle policies on replicated data  
- Monitoring usage and costs  

### Example: 1TB/month Replication
- Standard → Standard: ~$43  
- Standard → Standard-IA: ~$33 (23% savings)  
- Inter-region transfer: ~$20  
- **Total: ~$53–63/month**

---

## Performance Considerations

- **Latency:** 2–3 minutes typical replication  
- **Bandwidth:** CRR consumes inter-region bandwidth, affecting costs and app performance  
- **Scalability:** Must test with:
  - Large objects (multi-GB)  
  - High volume (1000s of objects)  
  - Multiple bucket pairs  

---

## Security & Compliance Impact

### Data Encryption
- Source encryption preserved  
- In-transit encryption used  
- Key management considerations for cross-region  

### Access Control
- IAM role permissions (least privilege recommended)  
- Cross-account replication possible (not tested)  
- Full audit logging enabled  

### Compliance Benefits
- Geographic data redundancy  
- Automated backups  
- Immutable replication logs  

---

## Production Implementation Recommendations

- **Monitoring & Alerting**
  - Replication failures  
  - Cross-region bandwidth costs  
  - Storage growth patterns  

- **Backup Strategy**
  - CRR = near real-time replication  
  - Still need traditional backups  

- **Network Architecture**
  - Consider VPC Peering / Transit Gateway / Direct Connect  
  - Monitor cross-region traffic  

- **Cost Management**
  - Lifecycle policies  
  - Regular reviews of replication rules  
  - Storage class optimization  

---

## Technical Insights

- **Replication Metadata** → preserved but new version IDs in destination  
- **Conflict Resolution** → last-write-wins (timestamp-based)  
- **Monitoring** → Replication metrics track status, errors, bandwidth  

---

## Lessons Learned

- CRR is powerful for **disaster recovery & global apps** but requires careful planning.  
- Replication timing must be factored into **RPO/RTO planning**.  
- Storage class optimization helps with cost management.  
- IAM roles require **least-privilege permissions** for security.  
- Automatic replication creates dependencies that must be **monitored & managed**.  

---
