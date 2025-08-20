# Lab 3: Delete Behavior in Cross-Region Replication (CRR)

## Overview

When testing delete operations with CRR, it’s important to understand that **deletes are not fully replicated by default**. AWS S3 treats deletes differently from uploads, depending on whether they are standard deletes (which create delete markers) or permanent deletes.

---

## Observed Delete Behaviors

1. **Standard Delete in Source (Delete Marker Created)**

   * In a **versioned bucket**, deleting an object without specifying a version creates a **delete marker**.
   * **Replication Behavior**: By default, **delete markers are *not* replicated** to the destination bucket.
   * **Result**: The deleted object still appears in the destination bucket.

2. **Permanent Delete of Specific Version**

   * If a specific object version is **permanently deleted** in the source bucket, CRR does **not replicate the deletion**.
   * The corresponding version in the destination remains intact.
   * This provides stronger durability and prevents accidental loss across both regions.

3. **Replication Rule Option (Delete Marker Replication)**

   * There is a **setting in CRR** called **“Delete marker replication”**.
   * If enabled, delete markers **are replicated** to the destination bucket, ensuring deletes appear consistent across both regions.
   * For this lab, the option was **not enabled**, so deletes only applied locally in the source bucket.

---

## Key Differences Documented

* **Uploads** → Replicated automatically to destination.
* **Delete markers (default delete)** → Not replicated unless explicitly enabled in rule.
* **Permanent version deletes** → Never replicated, destination keeps its copy.

---

**Best Practice**: In production, carefully decide whether to enable delete marker replication.

* **Enabled** → Keeps source and destination consistent.
* **Disabled** → Provides protection from accidental deletions by retaining data in destination.

---

