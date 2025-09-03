# AWS Web Server with Terraform

Simple web server deployment demonstrating Infrastructure as Code principles using Terraform and AWS.

## Architecture

- **EC2 Instance**: t2.micro (AWS Free Tier eligible)
- **Security Group**: Allows HTTP traffic on port 80
- **AMI**: Latest Amazon Linux 2023 (automatically selected)
- **Web Server**: Apache HTTP server with custom page

## Features

- Automatic AMI selection using data sources
- User data script for automated Apache installation
- Free tier optimized configuration
- Clean resource tagging
- Comprehensive outputs

## Prerequisites

- AWS CLI configured with valid credentials
- Terraform installed (v1.0+)
- AWS account with Free Tier available

## Usage

```bash
# Initialize Terraform
terraform init

# Preview changes
terraform plan

# Deploy infrastructure
terraform apply

# Access the web server using the output URL
# Example: http://YOUR_PUBLIC_IP

# Clean up resources
terraform destroy
