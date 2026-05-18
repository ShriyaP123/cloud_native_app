# Cloud Native Web Application

A production-grade cloud native application built on AWS, covering everything from infrastructure provisioning to CI/CD automation.

---

## Repositories

| Repository | Description | Tech |
|---|---|---|
| [webapp](https://github.com/ShriyaP123/webapp) | RESTful API with authentication, email verification, and file upload | Java, Spring Boot, MySQL |
| [tf-infra](https://github.com/ShriyaP123/tf-infra) | AWS infrastructure provisioned entirely with Terraform | Terraform, AWS |
| [serverless](https://github.com/ShriyaP123/serverless) | Lambda function for email verification via SNS trigger | AWS Lambda, SendGrid |

---

## Architecture Overview

```
                        ┌─────────────────────────────────────────────┐
                        │                   AWS VPC                    │
                        │                                              │
   HTTPS ──────────► ALB │──► Auto Scaling Group (EC2 + Spring Boot)  │
                        │              │                               │
                        │              ▼                               │
                        │         RDS (MySQL)    S3 (file storage)     │
                        │                                              │
                        └─────────────────────────────────────────────┘
                                       │
              User registers           ▼
              ──────────────► SNS Topic ──► Lambda ──► SendGrid ──► Email
                                       │
                              DynamoDB (dedup check)
```

---

## Infrastructure (tf-infra)

Provisioned entirely with Terraform across **DEV** and **DEMO** AWS accounts.

- **Networking** — VPC with public/private subnets across 3 availability zones
- **Compute** — Auto Scaling Group (min 3, max 5) with CPU-based scaling policies
- **Load Balancing** — Application Load Balancer with HTTPS termination
- **DNS** — Route 53 with subdomain delegation per environment
- **Security** — AWS KMS encryption for EC2, RDS, S3, and Secrets Manager; zero direct EC2 access
- **Certificates** — AWS ACM for dev, Namecheap-imported cert for demo

---

## Application (webapp)

RESTful API built with Spring Boot and backed by MySQL on AWS RDS.

- User registration and authentication (Basic Auth, BCrypt password hashing)
- Token-based email verification with expiry
- Profile picture upload to S3
- Custom StatsD metrics and CloudWatch logging
- CI/CD via GitHub Actions: run tests → build JAR → bake Packer AMI → trigger instance refresh

---

## Serverless (serverless)

AWS Lambda function triggered by SNS on user registration.

- Checks DynamoDB to prevent duplicate verification emails
- Sends a time-limited verification link via SendGrid
- Deployed via a separate GitHub Actions pipeline on every commit

---

## CI/CD Pipeline

```
Push to main
     │
     ▼
GitHub Actions
     ├── Run unit tests
     ├── Build JAR
     ├── Bake AMI with Packer (app + dependencies baked in)
     ├── Share AMI from DEV → DEMO account
     └── Trigger Auto Scaling Group instance refresh
              └── Wait for completion before marking deploy successful
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Application | Java 21, Spring Boot, Hibernate |
| Database | MySQL 8 on AWS RDS |
| Infrastructure | Terraform, AWS (VPC, EC2, RDS, S3, ALB, ASG) |
| Security | AWS KMS, Secrets Manager, ACM |
| Serverless | AWS Lambda, SNS, DynamoDB |
| Email | SendGrid, SPF + DKIM records |
| Observability | CloudWatch, StatsD |
| CI/CD | GitHub Actions, Packer |
| DNS | Route 53 |

---
