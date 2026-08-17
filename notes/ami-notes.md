# AWS AMI (Amazon Machine Image) Technical Notes & Interview Guide

A technical reference guide detailing Amazon Machine Image (AMI) concepts, golden image strategies, AMI lifecycle, cross-region replication, and interview preparation.

---

## 1. Core AMI Concepts

- **What is an AMI?**: An AMI is a master image template containing the software configuration (operating system, application server, pre-installed software, dependencies, and settings) required to launch an EC2 instance.
- **Components of an AMI**:
  1. One or more EBS snapshots (or template for instance-store backed root volume).
  2. Block device mapping specifying volume attachments.
  3. Launch permissions (Private, Public, or shared with specific AWS Account IDs).

---

## 2. AMI Types & Backing Mechanics

- **EBS-Backed AMIs**:
  - Root device is an EBS volume created from an EBS snapshot.
  - Instances can be stopped, restarted, and modified without losing data.
  - Fast boot time (< 1 minute).
- **Instance Store-Backed AMIs**:
  - Root device is an ephemeral instance store volume.
  - Instances cannot be stopped (only rebooted or terminated); data on root volume is lost upon termination.

---

## 3. Golden Image Pattern & Architecture

The **Golden Image** design pattern involves pre-building custom AMIs with security patches, monitoring agents (CloudWatch, Datadog), application code, and configurations pre-baked into the image.

### Advantages:
- **Fast Auto Scaling**: EC2 instances boot instantly without waiting for long user data scripts (`yum update`, package downloads).
- **Immutable Infrastructure**: Software releases are deployed as newly built AMIs rather than updating running servers in place.

---

## 4. Key Interview Questions & Answers

### Q1: Can an AMI registered in `us-east-1` be used directly to launch an EC2 instance in `eu-west-1`?
> **Answer**: No. AMIs are Region-scoped resources. To use an AMI in another Region, you must copy the AMI to the target Region (`aws ec2 copy-image`), which creates an AMI with a new unique AMI ID in that Region.

### Q2: What is the difference between an EBS Snapshot and an AMI?
> **Answer**: An EBS snapshot is a point-in-time raw block-level backup of a single EBS volume. An AMI is a higher-level template containing OS boot parameters, block device mappings for all attached volumes, and architectural definitions required to launch a functional EC2 instance.

### Q3: How do you automate AMI creation and updates in a CI/CD pipeline?
> **Answer**: Using tools like **HashiCorp Packer** or **EC2 Image Builder** integrated into a CI/CD pipeline (e.g., GitHub Actions, AWS CodePipeline) to automatically build, test, patch, and publish updated AMIs on a schedule.
