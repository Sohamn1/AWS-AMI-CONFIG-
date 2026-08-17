# AWS EBS Backup & Management Commands

This reference guide contains Linux terminal and AWS CLI commands used to manage Amazon Elastic Block Store (EBS) volumes, file systems, mount points, data persistence verification, and snapshot operations.

---

## 1. Volume & Disk Inspection Commands

Check existing storage blocks, attached disks, and mount points on the EC2 instance:

```bash
# List all block devices attached to the EC2 instance
lsblk

# View detailed file system disk space usage in human-readable format
df -h

# Check file system type of block devices (e.g., ext4, xfs)
sudo file -s /dev/xvd*
```

---

## 2. Formatting & Mounting EBS Volume

Format a newly attached raw EBS volume and mount it to a directory:

```bash
# Create a target mount directory
sudo mkdir -p /mnt/ebs-data

# Format raw EBS volume with ext4 file system (replace /dev/xvdf with actual device name)
sudo mkfs -t ext4 /dev/xvdf

# Mount the formatted EBS volume to the target directory
sudo mount /dev/xvdf /mnt/ebs-data

# Verify mount point configuration
df -h /mnt/ebs-data
```

---

## 3. Persistent Mount Configuration (`/etc/fstab`)

Ensure the volume automatically remounts upon instance reboot using UUID:

```bash
# Obtain the UUID of the attached EBS volume
sudo blkid /dev/xvdf

# Backup current fstab configuration
sudo cp /etc/fstab /etc/fstab.bak

# Append volume entry to /etc/fstab (replace UUID with actual UUID)
# UUID=xxxx-xxxx-xxxx-xxxx /mnt/ebs-data ext4 defaults,nofail 0 2
echo "UUID=$(sudo blkid -s UUID -o value /dev/xvdf) /mnt/ebs-data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab

# Test fstab entries without rebooting
sudo mount -a
```

---

## 4. Data Persistence & File Verification

Write test data to the EBS volume to confirm write permissions and persistence:

```bash
# Write sample application data to EBS volume
echo "EBS Data Backup Test - $(date)" | sudo tee /mnt/ebs-data/backup-sample.txt

# Verify written data content
cat /mnt/ebs-data/backup-sample.txt
```

---

## 5. AWS CLI Commands for EBS Operations

Perform EBS volume creation, attachment, and snapshot operations via AWS CLI:

```bash
# 1. Create a 10 GB gp3 EBS Volume in target Availability Zone
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=EBS-Backup-Volume}]'

# 2. Attach Volume to EC2 Instance
aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-0123456789abcdef0 \
  --device /dev/sdf

# 3. Create Point-in-Time EBS Snapshot
aws ec2 create-snapshot \
  --volume-id vol-0123456789abcdef0 \
  --description "Point-in-Time Backup Snapshot" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=EBS-Backup-Snapshot}]'

# 4. Describe Snapshots
aws ec2 describe-snapshots \
  --owner-ids self \
  --filters "Name=volume-id,Values=vol-0123456789abcdef0"
```
