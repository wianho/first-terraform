# Complete Terraform AWS Free Tier Tutorial - Updated Edition

## 📋 Table of Contents
1. [Prerequisites & Setup](#prerequisites--setup)
2. [AWS Account Configuration](#aws-account-configuration)
3. [Getting Your AWS Access Keys](#getting-your-aws-access-keys)
4. [Terraform Installation](#terraform-installation)
5. [Your First Terraform Project](#your-first-terraform-project)
6. [Understanding What We Built](#understanding-what-we-built)
7. [Testing and Validation](#testing-and-validation)
8. [Infrastructure Lifecycle Management](#infrastructure-lifecycle-management)
9. [Free Tier Safety](#free-tier-safety)
10. [Troubleshooting Common Issues](#troubleshooting-common-issues)
11. [Next Steps & Advanced Learning](#next-steps--advanced-learning)

---

## 📚 Prerequisites & Setup

### What You Need
- **AWS Account** (new accounts get 12 months free tier)
- **Terminal/Command Prompt** or AWS CloudShell access
- **Basic command line comfort**
- **30-60 minutes** for initial walkthrough

### Learning Objectives
By the end of this tutorial, you will:
- Understand Infrastructure as Code concepts
- Deploy real AWS infrastructure using Terraform
- Manage infrastructure lifecycle (create/update/destroy)
- Apply cost optimization and security best practices
- Have working knowledge to discuss in job interviews

---

## 🏗️ AWS Account Configuration

### Step 1: Create AWS Account
1. Go to [aws.amazon.com](https://aws.amazon.com)
2. Click **"Create an AWS Account"**
3. Complete signup with valid credit card (required but won't be charged if you stay in free tier)
4. Complete phone verification

### Step 2: Secure Your Account
**Enable Multi-Factor Authentication immediately:**
1. AWS Console → Click your account name (top right)
2. **Security credentials**
3. **Multi-factor authentication (MFA)** → Activate MFA
4. Use authenticator app or SMS

### Step 3: Set Up Billing Alerts
**Critical for free tier safety:**
1. AWS Console → **Billing and Cost Management**
2. **Billing preferences** → Enable all alert options
3. **Budgets** → Create budget:
   - Budget type: **Cost budget**
   - Amount: **$1.00**
   - Alert threshold: **$0.50** (50 cents)
   - Email alerts: Your email address

---

## 🔑 Getting Your AWS Access Keys

**This is where most beginners get stuck - here's the exact process:**

### Finding Your Access Keys

**Step 1: Navigate to Security Credentials**
1. Log into AWS Console at aws.amazon.com
2. Look at **top-right corner** - click on your account name/email
3. In dropdown menu, select **"Security credentials"**

**Step 2: Create Access Keys**
1. Scroll down to **"Access keys"** section
2. Click **"Create access key"**
3. **Warning about root user access:** For learning, click **"Continue"**
4. You'll see both keys displayed

**Step 3: Save Your Keys (CRITICAL)**
- **This is your ONLY chance to see the secret key**
- Click **"Download .csv file"** - saves to your computer
- Or copy both keys to a secure location
- **Don't close window** until you've saved everything

### What Your Keys Look Like
```
Access Key ID: AKIAIOSFODNN7EXAMPLE (starts with "AKIA")
Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY (long random string)
```

### Troubleshooting Key Access
**"I don't see Security Credentials"**
- Try searching "IAM" in the AWS console search bar
- Or click "My Account" instead of "Security credentials"

**"I closed the window before saving"**
- Delete the old access key and create a new one
- You can't recover a lost secret access key

**"AWS suggests IAM Identity Center"**
- For learning, ignore this and continue with traditional IAM
- Click "Continue to IAM" or "Maybe later"

---

## 🛠️ Terraform Installation

### AWS CloudShell (Recommended for Beginners)
If you have access to AWS CloudShell, this is the easiest option:

```bash
# Download and install Terraform
curl -O https://releases.hashicorp.com/terraform/1.9.8/terraform_1.9.8_linux_amd64.zip

# Install unzip if needed
sudo dnf install -y unzip

# Extract and install
unzip terraform_1.9.8_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify installation
terraform --version
```

### Local Installation

**Windows (using Chocolatey):**
```powershell
choco install terraform
```

**Mac (using Homebrew):**
```bash
brew install terraform
```

**Linux (Ubuntu/Debian):**
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```

**Verify Installation:**
```bash
terraform --version
# Should show: Terraform v1.x.x
```

---

## 🚀 Your First Terraform Project

### Configure AWS Access

**Test if already configured (CloudShell users):**
```bash
aws sts get-caller-identity
```

**If not configured, set up AWS CLI:**
```bash
aws configure
# Enter your Access Key ID
# Enter your Secret Access Key  
# Region: us-east-1
# Output: json
```

### Project Setup

```bash
# Create project directory
mkdir my-first-terraform
cd my-first-terraform
```

### Create Infrastructure Configuration

**main.tf:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# Automatically find the latest Amazon Linux 2023 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}

# Security group to allow HTTP traffic
resource "aws_security_group" "web_sg" {
  name        = "terraform-web-sg"
  description = "Security group for web server"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "terraform-web-sg"
  }
}

# Create EC2 instance
resource "aws_instance" "hello_terraform" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"  # Free tier eligible
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  # Automatic web server setup
  user_data = <<-USERDATA
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>🚀 Hello from Terraform!</h1><p>Server created: $(date)</p><p>Instance: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</p>" > /var/www/html/index.html
  USERDATA

  tags = {
    Name = "terraform-learning-server"
  }
}

# Outputs - show important information after deployment
output "server_public_ip" {
  description = "Public IP address of the web server"
  value       = aws_instance.hello_terraform.public_ip
}

output "server_url" {
  description = "URL to access the web server"
  value       = "http://${aws_instance.hello_terraform.public_ip}"
}
```

### Deploy Your Infrastructure

```bash
# 1. Initialize Terraform (downloads providers)
terraform init

# 2. Validate configuration
terraform validate

# 3. Preview what will be created
terraform plan

# 4. Create the infrastructure
terraform apply
# Type "yes" when prompted
```

### Expected Output
```
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

server_public_ip = "107.21.65.226"
server_url = "http://107.21.65.226"
```

---

## 🔍 Understanding What We Built

### Infrastructure Components

**Data Source (`aws_ami`):**
- Automatically finds the latest Amazon Linux 2023 AMI
- Eliminates hardcoded AMI IDs that become outdated
- Makes infrastructure portable across regions

**Security Group (`aws_security_group.web_sg`):**
- Acts as a virtual firewall
- Allows HTTP traffic (port 80) from anywhere
- Allows all outbound traffic
- Blocks SSH and other unnecessary ports

**EC2 Instance (`aws_instance.hello_terraform`):**
- t2.micro instance type (free tier eligible)
- Automatic web server installation via user data
- Associated with our security group
- Placed in default VPC and subnet

**Outputs:**
- Display important information after deployment
- Make it easy to access and test your infrastructure

### The Power of Infrastructure as Code

**What just happened:**
- 60 lines of declarative code became real infrastructure
- Automatic AMI selection ensures we always use latest images
- User data script configures the server without manual intervention
- Everything is documented, version-controlled, and reproducible

**This scales from 1 server to 10,000 servers** using the same principles.

---

## 🧪 Testing and Validation

### Test Your Web Server

```bash
# Test the web server (replace with your actual IP)
curl http://YOUR_SERVER_IP

# Check HTTP response headers
curl -I http://YOUR_SERVER_IP

# Expected response includes:
# HTTP/1.1 200 OK
# Server: Apache/2.4.x (Amazon Linux)
```

### Verify in AWS Console

**EC2 Dashboard:**
- Navigate to EC2 → Instances
- Find "terraform-learning-server"
- Verify it's in "running" state

**Security Groups:**
- EC2 → Security Groups
- Find "terraform-web-sg"
- Verify HTTP (port 80) is allowed inbound

**Cost Verification:**
- Billing → Free Tier Usage
- Confirm EC2 hours are within 750-hour limit

---

## 🔄 Infrastructure Lifecycle Management

### Making Changes

**Example: Update the web page content**
```bash
# Edit main.tf - modify the user_data section
# Change the HTML content in the echo command
terraform plan   # See what will change
terraform apply  # Apply changes
```

### Viewing Current State

```bash
# Show all resources in current state
terraform show

# List all managed resources
terraform state list

# Get details about specific resource
terraform state show aws_instance.hello_terraform
```

### Infrastructure Destruction

**When finished learning:**
```bash
terraform destroy
# Type "yes" when prompted

# This will:
# - Terminate the EC2 instance
# - Delete the security group
# - Remove all created resources
# - Reset AWS account to previous state
```

**Verify cleanup:**
```bash
# Check that resources are gone
aws ec2 describe-instances --query 'Reservations[*].Instances[?Tags[?Key==`Name` && Value==`terraform-learning-server`]]'
# Should return empty: []
```

---

## 💰 Free Tier Safety

### What's Included in Free Tier

**EC2 (12 months):**
- 750 hours/month of t2.micro or t3.micro instances
- 30 GB of EBS storage
- 15 GB of bandwidth out per month
- 2 million I/O operations

**Always Free Services:**
- VPC, subnets, security groups, internet gateways
- Route tables and basic networking
- CloudWatch basic monitoring

### Our Configuration Cost Analysis

```
✅ t2.micro instance:     $0.00 (Free tier covers 750 hours/month)
✅ Security group:        $0.00 (Always free)
✅ Default VPC usage:     $0.00 (Always free)
✅ Basic monitoring:      $0.00 (Basic CloudWatch included)
✅ 8 GB EBS storage:      $0.00 (Under 30 GB free tier limit)
✅ Data transfer:         $0.00 (Under 15 GB free tier limit)
-----------------------------------------------------------
Total Monthly Cost:       $0.00
```

### Staying Within Free Tier

**Safe Practices:**
- Only use t2.micro or t3.micro instance types
- Run `terraform destroy` when not actively learning
- Monitor usage in AWS Console → Billing → Free Tier
- Set up billing alerts at $1 threshold

**Avoid These Costly Mistakes:**
- t2.small or larger instances (immediate charges)
- Multiple instances running simultaneously
- Application Load Balancers (~$18/month)
- NAT Gateways (~$32/month)
- Elastic IP addresses when instances are stopped

---

## 🔧 Troubleshooting Common Issues

### AWS Credentials Issues

**"Unable to locate credentials"**
```bash
# Check if AWS CLI is configured
aws sts get-caller-identity

# If not working, reconfigure
aws configure
```

**"Access Denied" errors**
- Verify your access keys are correct
- Ensure your AWS account has necessary permissions
- Check if you're using the correct region

### Terraform Issues

**"AMI not found" errors**
- Our configuration uses data sources to automatically find AMIs
- This should not occur, but if it does, check your region settings

**"Resource already exists"**
```bash
# If you have naming conflicts
terraform import aws_security_group.web_sg sg-xxxxxxxxx
# Or change the resource names in your configuration
```

**"State file locked"**
```bash
# If Terraform was interrupted
terraform force-unlock LOCK_ID
```

### Infrastructure Issues

**Can't connect to web server**
- Wait 2-3 minutes after `terraform apply` for full boot
- Verify security group allows HTTP (port 80)
- Check AWS Console that instance is "running"
- Try HTTP, not HTTPS (we don't have SSL configured)

**Instance keeps stopping**
- Check AWS Free Tier usage limits
- Verify you're using t2.micro instance type
- Check for any billing issues

---

## 📈 Next Steps & Advanced Learning

### Week 1-2: Master the Basics
- [ ] Practice the full lifecycle: init, plan, apply, destroy
- [ ] Modify HTML content and redeploy
- [ ] Experiment with different instance types (staying in free tier)
- [ ] Add SSH key pairs for server access

### Week 3-4: Expand Your Infrastructure
- [ ] Add multiple servers
- [ ] Create custom VPC with public/private subnets
- [ ] Implement Application Load Balancer
- [ ] Add RDS database instance

### Month 2: Advanced Concepts
- [ ] Terraform modules for reusable code
- [ ] Remote state storage in S3
- [ ] Multiple environments (dev/staging/prod)
- [ ] Integration with CI/CD pipelines

### Month 3: Professional Skills
- [ ] Terraform best practices and conventions
- [ ] Security hardening and compliance
- [ ] Monitoring and alerting with CloudWatch
- [ ] Cost optimization strategies

### Portfolio Projects for Job Interviews

**Project 1: Multi-Tier Web Application**
- Web servers behind load balancer
- RDS database with backup strategy
- Auto Scaling Groups
- Route53 DNS management

**Project 2: Microservices Infrastructure**
- Container orchestration with ECS
- Service discovery and load balancing
- Centralized logging and monitoring
- Blue-green deployment capability

**Project 3: Hybrid Cloud Setup**
- VPN connections to on-premises
- Directory service integration
- Compliance and security controls
- Disaster recovery procedures

---

## 🎯 Key Takeaways for Your Career

### What You've Learned
- **Infrastructure as Code** fundamentals
- **AWS core services** and networking
- **Terraform workflow** and best practices
- **Cost management** and free tier optimization
- **Security basics** with proper firewall rules
- **Automation** through user data scripts

### How This Applies to $200k Jobs
- **DevOps Engineer roles** require exactly these skills
- **Site Reliability Engineer** positions value infrastructure automation
- **Cloud Engineer** roles expect Terraform proficiency
- **Platform Engineer** roles need infrastructure scaling knowledge

### Interview Talking Points
- "I can deploy reproducible infrastructure using Infrastructure as Code"
- "I understand AWS networking, security groups, and compute services"
- "I practice cost optimization and resource lifecycle management"
- "I've automated server configuration and deployment processes"

### Professional Differentiation
Your military background + Terraform skills + cost consciousness + systematic approach = exactly what companies need for senior infrastructure roles.

You now have hands-on experience with tools and concepts that justify the salary ranges you're targeting in Seattle.

---

## 📚 Additional Resources

**Official Documentation:**
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS Free Tier Details](https://aws.amazon.com/free/)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)

**Learning Platforms:**
- [HashiCorp Learn](https://learn.hashicorp.com/terraform)
- [AWS Training and Certification](https://aws.amazon.com/training/)
- [A Cloud Guru](https://acloudguru.com/)

**Community Resources:**
- [Terraform Community Forum](https://discuss.hashicorp.com/c/terraform-core/)
- [r/Terraform](https://reddit.com/r/Terraform)
- [AWS subreddit](https://reddit.com/r/aws)

---

**Remember:** Infrastructure as Code is not just about tools - it's a methodology that eliminates manual processes, ensures consistency, and enables rapid scaling. The concepts you've learned here apply across all cloud providers and form the foundation of modern infrastructure engineering.

**You're now equipped with skills that directly apply to the high-paying DevOps and Cloud Engineering roles you're targeting. Keep practicing, keep building, and keep automating!**
