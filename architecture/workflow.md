# AWS EBS & AMI Backup Workflow

This document describes the architecture, step-by-step technical workflow, and disaster recovery lifecycle for configuring an AWS EC2 Apache Web Server, creating EBS root snapshots, building custom AMIs, and launching cloned instances.

---

## 1. End-to-End Workflow Overview

```
[ Primary EC2 Instance ] (Apache Web Server)
          │
          ▼
   [ EBS Root Volume ] (Block Storage containing OS & Web Data)
          │
          ▼
   [ EBS Snapshot ] (Point-in-Time Incremental Backup)
          │
          ▼
    [ Custom AMI ] (Golden Image Template)
          │
          ▼
 [ New EC2 Instance ] (Provisioned from Custom AMI)
          │
          ▼
 [ Apache Web Server ] (Serves "Hello from Custom AMI")
```

---

## 2. Technical Workflow Steps

### Step 1: EC2 Instance & Web Server Provisioning
- Launch an EC2 instance (`t2.micro` / Amazon Linux 2023).
- Attach standard EBS Root Volume storing the Linux OS and web data directory (`/var/www/html`).
- Install and enable Apache HTTP server (`httpd`).
- Deploy application index page serving `"Hello from Custom AMI"`.

### Step 2: EBS Volume Backup (Snapshot)
- Create a point-in-time snapshot of the EBS Root Volume.
- AWS copies block data directly to Amazon S3 (managed by AWS, incremental backup).
- Captures system state, configurations, and persistent web application data.

### Step 3: Custom AMI Registration
- Create a Custom Amazon Machine Image (AMI) from the EC2 instance / EBS Snapshot.
- The Custom AMI bundles the EBS root snapshot along with block device mapping instructions, boot sector details, and kernel registration.

### Step 4: Instance Replication & Recovery
- Launch a brand new EC2 instance directly using the Custom AMI ID.
- The new instance automatically provisions its own EBS root volume cloned from the AMI snapshot.

### Step 5: Web Application Verification
- On boot, the new EC2 instance starts up with Apache pre-installed and configured.
- Navigating to the new instance's Public IP immediately serves `"Hello from Custom AMI"` without requiring manual script execution or configuration.

---

## 3. Key Operational Benefits

1. **Disaster Recovery (DR)**: Rapid recovery from instance failures by launching pre-configured replacements in minutes.
2. **Infrastructure Scaling**: Use the Custom AMI as a "Golden Image" in Auto Scaling Groups (ASG) behind an Application Load Balancer (ALB).
3. **Consistency & Reliability**: Eliminates configuration drift across development, staging, and production environments.
