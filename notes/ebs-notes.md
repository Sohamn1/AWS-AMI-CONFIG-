# AWS EBS (Elastic Block Store) Technical Notes & Interview Guide

A comprehensive summary of Amazon EBS concepts, volume types, snapshot mechanics, data persistence, and common technical interview questions.

---

## 1. Core EBS Concepts

- **Block Storage**: EBS provides persistent block-level storage volumes for use with EC2 instances. Each volume behaves like a raw, unformatted physical hard drive.
- **Availability Zone (AZ) Scope**: An EBS volume is strictly locked to a single Availability Zone. To move an EBS volume across AZs or Regions, you must create a snapshot and restore it in the target AZ/Region.
- **Persistence**: Unlike Instance Store volumes (ephemeral storage), data on an EBS volume persists independently of the EC2 instance life cycle.

---

## 2. EBS Volume Types Quick Reference

| Volume Type | API Name | Use Case | Max IOPS | Max Throughput |
| :--- | :--- | :--- | :--- | :--- |
| **General Purpose SSD** | `gp3` / `gp2` | System boot volumes, dev/test, web servers | 16,000 IOPS | 1,000 MB/s |
| **Provisioned IOPS SSD** | `io2` / `io1` | High-performance databases (I/O intensive) | 64,000 IOPS | 1,000 MB/s |
| **Throughput Optimized HDD** | `st1` | Big data, data warehouses, log processing | 500 IOPS | 500 MB/s |
| **Cold HDD** | `sc1` | Low-cost storage for infrequently accessed data | 250 IOPS | 250 MB/s |

---

## 3. EBS Snapshots Deep Dive

- **Point-in-Time Incremental Backups**: The initial snapshot copies the entire volume. Subsequent snapshots only copy blocks that have changed since the last snapshot.
- **Stored in S3**: Snapshots are stored internally in Amazon S3 (managed by AWS, not visible in your S3 bucket list), providing 99.999999999% (11 9's) durability.
- **Crash Consistency vs. Application Consistency**:
  - For OS/data disk consistency, freeze file system I/O or detach volume before taking a snapshot if possible.
  - Multi-volume crash-consistent snapshots can be created using AWS CLI/Console.

---

## 4. Key Interview Questions & Answers

### Q1: How do you attach an EBS volume to an EC2 instance in another Availability Zone?
> **Answer**: EBS volumes cannot be directly attached across AZs. You must first take an EBS snapshot of the original volume, and then create a new volume from that snapshot specifying the target Availability Zone.

### Q2: What happens to the EBS root volume when an EC2 instance is terminated?
> **Answer**: By default, the `DeleteOnTermination` attribute is set to `true` for the root volume, so it gets deleted automatically. Non-root attached volumes have `DeleteOnTermination` set to `false` by default and persist after instance termination.

### Q3: How do EBS snapshots save storage costs?
> **Answer**: EBS snapshots are incremental. Only modified data blocks are saved, drastically reducing S3 storage overhead compared to full backups each time.
