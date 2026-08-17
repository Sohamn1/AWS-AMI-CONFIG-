# AWS EC2 Persistent Storage & Custom AMI Project

A hands-on project demonstrating EBS volume management, snapshot backups, and custom AMI creation on AWS EC2 — simulating a real-world "golden image" provisioning workflow.

---

## Overview

This project covers three core AWS concepts that Cloud Support and Junior DevOps roles rely on daily:

1. **EBS (Elastic Block Store)** — adding persistent storage to an EC2 instance
2. **EBS Snapshots** — point-in-time backups of volumes
3. **Custom AMI** — cloning a fully configured instance (OS + Apache + website) for repeatable deployment

---

## Part 1 — EBS (Elastic Block Store)

Goal: attach persistent, durable storage to an EC2 instance and prove the data survives independently of the instance's root volume.

**Steps:**
- Launched an EC2 instance with Apache (`httpd`) installed
- Created a new EBS volume
- Attached the volume to the running instance
- Connected via SSH
- Formatted the volume with the XFS filesystem
- Mounted it at `/data`
- Created a test file and verified persistence after remounting

**Commands used:**
```bash
lsblk
mkfs -t xfs /dev/xvdf
mkdir /data
mount /dev/xvdf /data
echo "Hello from EBS" | sudo tee /data/test.txt
cat /data/test.txt
```

---

## Part 2 — EBS Snapshot

Goal: create a reliable, point-in-time backup of the EBS volume.

**Steps:**
- Created a snapshot of the attached volume
- Waited for the snapshot status to reach `Completed`
- Captured the Snapshot ID and metadata for reference

---

## Part 3 — Custom AMI

This was the core of the project — proving that a fully configured server (OS, packages, files, config) can be captured and redeployed identically.

### AMI Creation
- Created a custom AMI from the configured EC2 instance
- Waited until the AMI status became `Available`

### Launch from AMI
- Launched a new EC2 instance from the custom AMI
- Connected via SSH:
```bash
ssh -i "soham.pem" ec2-user@<public-ip>
```

### Verification

Confirmed the webpage persisted from the original instance:
```bash
cat /var/www/html/index.html
```
**Output:**
```html
<h1>Hello from Custom AMI</h1>
```

Confirmed Apache was running out of the box, with no reconfiguration:
```bash
sudo systemctl status httpd
```
**Status:** `Active: active (running)`

This confirms the AMI successfully copied:
- Operating System
- Apache installation
