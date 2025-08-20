# MFA Delete Documentation

## Overview

MFA (Multi-Factor Authentication) Delete is an additional security feature for Amazon S3 buckets with versioning enabled. It provides an extra layer of protection by requiring MFA authentication for certain high-risk operations.

---

## Requirements

* **Root User Only**:
  MFA Delete can only be enabled or disabled by the **AWS account root user**.
* **AWS CLI with MFA Configuration**:
  Must use the AWS CLI and provide valid MFA configuration (MFA device + token) when enabling.

---

## Operations Requiring MFA Token

Once MFA Delete is enabled, an MFA code is **mandatory** for the following operations:

* **Permanently deleting object versions** (using `s3api delete-object` with `--version-id`)
* **Suspending bucket versioning** (changing from `Enabled` to `Suspended`)

---

## Operations Not Requiring MFA

* **Regular delete operations** (e.g., `aws s3 rm`) only add a **delete marker** to the latest version of the object.
  The object version itself is not permanently removed and does **not** require MFA.

---

Best Practice: Use MFA Delete for critical buckets to prevent accidental or malicious permanent deletions.

---


