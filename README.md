# AWS 3-Tier Infrastructure with CI/CD Automation using Terraform and Jenkins

## Project Overview

A production-style DevOps project where infrastructure provisioning and application deployment are automated using Terraform and Jenkins.

Terraform provisions all AWS infrastructure. Jenkins runs the CI/CD pipeline. 
The Node.js application is deployed to EC2 instances behind an 
Application Load Balancer with Auto Scaling.

## Architecture

Traffic Flow:

User
8+ ↓
Internet Gateway
 ↓
Application Load Balancer (Public Subnet)
 ↓
Auto Scaling Group — EC2 (Private Subnet)

CI/CD + State Flow:

Jenkins
 ↓
S3 (Artifact Storage)
 ↓
Instance Refresh → New EC2 pulls artifact from S3

Terraform Remote State → S3 bucket + DynamoDB (state lock)

## Tech Stack

| Category        | Tool                                        |
|-----------------|---------------------------------------------|
| Cloud           | AWS (EC2, VPC, S3, IAM, ALB, Auto Scaling)  |
| IaC             | Terraform                                   |
| CI/CD           | Jenkins (Docker-based build environment)    |
| App             | Node.js                                     |
| Scripting       | Bash (user-data)                            |
| Version Control | Git & GitHub                                |

## Repository Structure

```
├── app/               # Node.js application
├── bootstrap/         # Terraform config for S3 remote state bucket and DynamoDB state lock table
├── jenkins/           # Jenkinsfile and pipeline config
├── terraform/
│   ├── main.tf        # All infrastructure resources
│   ├── backend.tf     # S3 remote state config
│   └── variables.tf   # Input variables
└── .gitignore
```


## CI/CD Workflow

1. Code pushed to GitHub
2. Jenkins pipeline triggered
3. App packaged as a zip artifact
4. Artifact uploaded to S3 (versioned)
5. Latest artifact copied as `node-app-latest.zip` in S3
6. Auto Scaling instance refresh triggered
7. New EC2 instances launch and pull artifact from S3 via user-data script
8. ALB routes traffic to healthy instances only

## Key Features

- Infrastructure as Code using Terraform
- Remote state stored in S3 with locking via backend config
- Secure architecture — EC2 in private subnets, no direct SSH
- IAM roles for EC2 to access S3 (no hardcoded credentials)
- Auto Scaling for high availability
- Rollback via S3 versioning — redeploy previous artifact version
- Pipeline workspace cleaned after every run

## Challenges Faced

- IAM S3 access denied — fixed by correcting EC2 instance profile policy
- Target group showing unhealthy instances — resolved health check path mismatch
- Instance refresh not triggering — fixed ASG name reference in Jenkins pipeline
- Terraform state issues — resolved by configuring S3 backend correctly

## Future Improvements

- Add GitHub webhook to auto-trigger Jenkins pipeline on code push
- Containerize app with Docker, push to ECR
- Deploy on Kubernetes (EKS)
- Add multi-environment setup (Dev / Stage / Prod)
- Add CloudWatch monitoring and alerting
- Use custom Docker image with aws-cli pre-installed to speed up pipeline
