# Project-1-Scalable-Web-Application-with-ALB-and-Auto-Scaling
# AWS 2-AZ Web Application Architecture (VPC + ASG + RDS + CloudWatch + SNS)

This project documents a highly available AWS reference architecture deployed across **two Availability Zones**. It uses a **VPC** with **public** and **private** subnets, an **Auto Scaling Group (ASG)** for the application tier, **Amazon RDS** in a dedicated database subnet group, and **Amazon CloudWatch + SNS** for monitoring and alert notifications.

---

## Architecture Overview

The design follows a common best-practice pattern:

- **Public subnets (per AZ)** host **NAT Gateways** (and typically an internet-facing **Load Balancer**).
- **Private subnets (per AZ)** host the **application EC2 instances** managed by an **Auto Scaling Group**.
- **DB subnets (private, per AZ)** host **Amazon RDS** (Multi-AZ capable).
- **CloudWatch** collects metrics/logs and triggers alarms.
- **SNS** sends notifications to subscribers such as email.
---

## Components

### Networking
- **Region**
- **VPC**
- **2 Availability Zones**
- **Public Subnets (AZ-a, AZ-b)**
  - NAT Gateway (one per AZ recommended)
- **Private Subnets (AZ-a, AZ-b)**
  - Application instances (EC2)
- **DB Subnet Group**
  - Private DB subnets across both AZs for RDS placement
- **Internet Gateway (IGW)**

### Compute
- **Auto Scaling Group (ASG)**
  - Spans private subnets in both AZs
  - Scales in/out based on CloudWatch alarms or policies

### Database
- **Amazon RDS**
  - Deployed in private DB subnets
  - Optional Multi-AZ for high availability (primary/standby)

### Observability & Alerts
- **Amazon CloudWatch**
  - Metrics, logs, dashboards, alarms
- **Amazon SNS**
  - Notification topic for alarms
  - Subscribers: Email endpoint

---

## Traffic Flow

### Inbound 
1. User traffic enters the VPC via the **Internet Gateway (IGW)**
2. Traffic terminates on an **Application Load Balancer (ALB)** in the **public subnets**
3. The ALB forwards requests to **EC2 instances** in the **private subnets**
4. EC2 instances connect to **Amazon RDS** in the **DB subnets**

### Outbound (private subnets)
- Private subnets route outbound traffic (package updates, external APIs) through:
  **Private Subnet → NAT Gateway (public subnet) → IGW → Internet**

---

## Routing Tables

### Public Route Table
- `0.0.0.0/0 → Internet Gateway (IGW)`

### Private Route Table
- `0.0.0.0/0 → NAT Gateway (same AZ preferred)`

### DB Subnets
- No direct internet route
- Only allow necessary access from the application tier

---

## Security

Use **Security Groups** to tightly control traffic:

- **ALB Security Group**
  - Inbound: `80/443` from the internet (or allowed CIDRs)
  - Outbound: to application SG on app port(s)

- **Application Security Group**
  - Inbound: app port(s) **only from ALB SG**
  - Outbound: to DB SG on DB port and to internet via NAT as required

- **RDS Security Group**
  - Inbound: DB port **only from App SG**

---

## CloudWatch & SNS
### Alerting Path
- **CloudWatch Alarm → SNS Topic → Subscriber(s)**  
Examples of subscribers:
- Email/SMS to on-call team
- HTTPS endpoint (Slack/PagerDuty integration)
- Lambda for automated remediation

---
