# AWS S3 Lab 1 Report: Storage Classes & Lifecycle Policies  

**Name:** Cletus-Nehinlalei-Mangu  
**Course:** DevOps Training - Solutions Architect Associate Preparation  

---

## Lab Summary  
In this lab, I worked with Amazon S3 storage classes and lifecycle policies to better understand how data can be stored efficiently and moved between tiers to save costs. I created a bucket, uploaded files into different storage classes, and then set up lifecycle rules that automatically transition objects based on their age and usage. This gave me hands-on practice with cost optimization and real-world storage management strategies.  

---

## Lab Objectives Met  
Here’s what I achieved in the lab:  

- Created the S3 bucket **cnm-lab1-lifecycle-20250819** in the **us-east-1** region.  
- Uploaded three test files using different storage classes: **Standard**, **Standard-IA**, and **Intelligent-Tiering**.  
- Configured a lifecycle policy with multiple transition stages.  
- Analyzed how costs vary across different storage tiers.  

---

## What Worked Well  

- **Bucket creation:** Straightforward through the AWS Console. Public access was blocked by default, ACLs were disabled, and permissions worked without issues.  

- **Storage classes:**  
  1. `test1.txt` → **Standard** (immediate access, no retrieval fees)  
  2. `test2.txt` → **Standard-IA** (30-day minimum storage requirement, early deletion fees)  
  3. `test3.txt` → **Intelligent-Tiering** (automatic optimization based on access patterns)  

- **Lifecycle policy:** Configured to transition data at specific intervals:  
  1. **30 days** → Standard-IA (46% cheaper)  
  2. **90 days** → Glacier Flexible Retrieval (68% cheaper)  
  3. **365 days** → Glacier Deep Archive (95% cheaper)  
  4. **2,555 days** → Expiration for compliance purposes  

---

## Challenges I Faced  

- **Storage class warnings:** Uploading to Standard-IA displayed warnings about early deletion fees and 30-day minimum storage duration. This highlighted that costs involve more than just per-GB pricing.  
- **Cost complexity:** Savings depend on multiple factors such as retrieval frequency, Intelligent-Tiering monitoring fees, and minimum duration charges.  
- **Interface navigation:** The lifecycle rule setup was hidden under the *Management* tab, which wasn’t obvious at first. The many configuration options also required careful consideration.  

---

## Real-World Application Scenarios  

- **Document management systems:**  
  - Active files stay in Standard.  
  - Archives move to Standard-IA.  
  - Long-term records move to Glacier.  
  - Temporary files expire automatically.  

- **Media companies:**  
  - Current video projects remain in Standard.  
  - Completed projects transition to Standard-IA.  
  - Old archives move to Glacier.  
  - Expiration rules manage temporary rendering files.  

- **Backups and disaster recovery:**  
  - Recent backups in Standard-IA for quick recovery.  
  - Monthly backups transition to Glacier.  
  - Compliance archives stored in Deep Archive.  
  - Automated cleanup removes outdated versions.  

---

## Cost Implications Analysis  

- **Storage pricing in us-east-1:**  
  1. **Standard:** $0.023/GB/month  
  2. **Standard-IA:** $0.0125/GB/month (46% savings, retrieval fees apply)  
  3. **Glacier Flexible Retrieval:** $0.0036/GB/month (84% savings, hours for retrieval)  
  4. **Glacier Deep Archive:** $0.00099/GB/month (96% savings, up to 12 hours retrieval)  

---
