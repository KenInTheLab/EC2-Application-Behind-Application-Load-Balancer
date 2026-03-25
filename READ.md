## Watch me Build Here!
https://www.loom.com/share/8f266b0d072e4f9a914fd7e627f7446b

# Project 03 — EC2 App Behind an Application Load Balancer

![AWS](https://img.shields.io/badge/AWS-ALB%20%7C%20EC2-orange?logo=amazonaws)
![Terraform](https://img.shields.io/badge/IaC-Terraform-7B42BC?logo=terraform)
![Status](https://img.shields.io/badge/Status-Live%20Demo-brightgreen)

## Overview

Deploys two EC2 web servers behind an AWS Application Load Balancer using Terraform. Traffic is distributed across both instances with automatic health checks. EC2 instances are isolated from direct internet access — only the ALB can reach them. This is the core high-availability pattern used in real production AWS environments.

## Architecture

```
Browser
   │
   ▼
Application Load Balancer (public, port 80)
   │                  │
   ▼                  ▼
EC2 Instance 1     EC2 Instance 2
(Apache · AZ-1)    (Apache · AZ-2)
        │                  │
        └──── Target Group ┘
              (health checks)
```

Terraform provisions:
- ALB security group (internet → port 80)
- EC2 security group (ALB only → port 80, no direct public access)
- Two EC2 instances in separate subnets across multiple AZs
- Application Load Balancer with HTTP listener
- Target group with health checks
- Target group attachments for both instances

## AWS Services Used

| Service | Purpose |
|---------|---------|
| Amazon EC2 | Web server instances (x2) |
| Application Load Balancer | Traffic distribution + health checks |
| VPC Security Groups | ALB isolation pattern |
| Target Group | Instance registration + health monitoring |

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) >= 1.3.0
- [AWS CLI](https://aws.amazon.com/cli/) configured (`aws configure`)
- AWS account with default VPC containing at least 2 subnets

## Repo Structure

```
project-03-alb-ec2/
├── main.tf          # ALB, listener, target group, 2x EC2, security groups
├── variables.tf     # Region, instance type, tags
├── outputs.tf       # ALB DNS URL, instance IDs
├── provider.tf      # AWS provider + Terraform version
└── README.md
```

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/project-03-alb-ec2
cd project-03-alb-ec2

# 2. Deploy
terraform init
terraform plan
terraform apply -auto-approve

# 3. Wait ~3 minutes for ALB to become active and health checks to pass
# Then open the ALB DNS URL from the output

# 4. Refresh the browser multiple times
# You will see Server 1 and Server 2 alternating — load balancing in action
```

## Outputs

| Output | Description |
|--------|-------------|
| `alb_dns_name` | Load balancer URL (use this to access the app) |
| `instance_1_id` | EC2 Instance 1 ID |
| `instance_2_id` | EC2 Instance 2 ID |

## Cost

| Resource | Monthly Cost |
|----------|-------------|
| 2x EC2 t2.micro | Free tier (750 hrs combined) |
| Application Load Balancer | ~$0.02/hr (~$0.50 for a demo session) |
| **Total** | **< $1.00 for demo** |

> Destroy immediately after recording your demo to avoid ALB charges.

## Cleanup

```bash
terraform destroy -auto-approve
```

## Security Design Note

EC2 instances are not exposed directly to the internet. The EC2 security group only accepts inbound traffic from the ALB security group — not from the public CIDR. This is a standard production security pattern that prevents backend bypass.

## What This Demonstrates

- High-availability architecture on AWS
- Application Load Balancer provisioning with Terraform
- Security group chaining (ALB SG → EC2 SG)
- Multi-AZ instance deployment
- Target group health checks
- Real traffic distribution visible in the browser

## Production Improvements

- Add Auto Scaling Group (ASG) behind the target group
- Add HTTPS listener with ACM certificate
- Restrict access with WAF or IP allowlist
- Store Terraform state in S3 + DynamoDB

---

*Built as part of a Cloud Engineer portfolio — provisioned 100% with Terraform.*
