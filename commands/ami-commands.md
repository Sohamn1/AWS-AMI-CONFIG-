# AWS AMI Management & Instance Replication Commands

This reference guide details commands for installing and configuring an Apache Web Server (`httpd`), creating a Custom Amazon Machine Image (AMI), launching new EC2 instances from the custom AMI, and verifying web application deployment.

---

## 1. Apache Web Server Setup (`httpd`)

Install and start Apache web server on Amazon Linux 2 / Amazon Linux 2023:

```bash
# Update package index
sudo yum update -y

# Install Apache Web Server (httpd)
sudo yum install -y httpd

# Start Apache HTTP service
sudo systemctl start httpd

# Enable Apache service to start automatically on system boot
sudo systemctl enable httpd

# Verify Apache service status
sudo systemctl status httpd
```

---

## 2. Web Application Setup

Deploy custom HTML landing page served by Apache:

```bash
# Create custom HTML content
echo "<h1>Hello from Custom AMI</h1><p>Server provisioned via custom AMI backup.</p>" | sudo tee /var/www/html/index.html

# Verify local web server response
curl http://localhost
```

---

## 3. Creating Custom AMI via AWS CLI

Create an Amazon Machine Image (AMI) from the configured EC2 instance:

```bash
# Create Custom AMI from running EC2 instance
aws ec2 create-image \
  --instance-id i-0123456789abcdef0 \
  --name "Custom-Apache-AMI-$(date +%Y%m%d)" \
  --description "Golden AMI with pre-configured Apache web server" \
  --no-reboot \
  --tag-specifications 'ResourceType=image,Tags=[{Key=Name,Value=Custom-Apache-AMI}]'

# Check AMI creation status (Pending -> Available)
aws ec2 describe-images \
  --owners self \
  --filters "Name=name,Values=Custom-Apache-AMI*" \
  --query "Images[*].{ID:ImageId,State:State,Name:Name}" \
  --output table
```

---

## 4. Launching New EC2 Instance from Custom AMI

Deploy a new EC2 instance pre-loaded with Apache and website content using the Custom AMI:

```bash
# Launch new EC2 instance from Custom AMI ID
aws ec2 run-instances \
  --image-id ami-0123456789abcdef0 \
  --count 1 \
  --instance-type t2.micro \
  --key-name my-ec2-key \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=EC2-From-Custom-AMI}]'

# Query Public IP of newly launched instance
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=EC2-From-Custom-AMI" \
  --query "Reservations[*].Instances[*].{ID:InstanceId,PublicIP:PublicIpAddress,State:State.Name}" \
  --output table
```

---

## 5. Verification & Testing

Verify that the new instance successfully serves the webpage without manual server setup:

```bash
# Test HTTP response using curl with the new instance's Public IP
curl http://<PUBLIC_IP>

# Expected output:
# <h1>Hello from Custom AMI</h1><p>Server provisioned via custom AMI backup.</p>
```
