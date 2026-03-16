# AID HealthTech – AWS Cloud Architecture

## Document Information
**Project:** AID HealthTech Platform  
**Document Type:** Cloud Architecture Documentation  
**Version:** 1.0  
**Author:** AID HealthTech Engineering  
**Last Updated:** 2026  

---

# 1. Overview

This document describes the cloud infrastructure architecture of the **AID HealthTech platform** running on **Amazon Web Services (AWS)**.

The architecture was designed with the following principles:

- High availability
- Scalability
- Security by design
- Observability
- Cost awareness
- Infrastructure isolation between environments

The system supports two operational environments:

- **Development (DEV)**
- **Production (PROD)**

Each environment is isolated and optimized according to its operational needs.

---

# 2. Architecture Layers

The platform follows a layered architecture model:

Users  
Edge Layer  
Application Layer  
Data Layer  
Shared Platform Services  
Security & Observability  

Request flow:

Users → CloudFront → WAF → Application Load Balancer → ECS Services → Data Layer

Static content flow:

Users → CloudFront → S3 Static Hosting

---

# 3. Edge Layer

## Amazon CloudFront

CloudFront acts as the global edge distribution layer.

Responsibilities:

- Content Delivery Network (CDN)
- TLS termination
- Static asset caching
- API traffic routing
- Performance optimization

---

## AWS Web Application Firewall (WAF)

AWS WAF protects the platform against common web attacks.

Security protections include:

- SQL injection prevention
- Cross-site scripting protection
- Rate limiting
- Bot filtering
- IP filtering rules

Flow:

CloudFront → WAF → Application Load Balancer

---

# 4. Frontend Layer

## Amazon S3 – Static Website Hosting

The frontend application is hosted as a static website in Amazon S3.

Stored assets include:

- HTML
- JavaScript bundles
- CSS
- Images
- Static files

CloudFront serves these assets globally.

Deployment is performed through the CI/CD pipeline.

---

# 5. Networking Architecture

Each environment operates inside its own **Amazon Virtual Private Cloud (VPC)**.

## VPC Components

Each VPC includes:

- Internet Gateway
- Route Tables
- Security Groups
- Subnets

Subnet separation:

Public Subnet  
Private Application Subnet  
Private Database Subnet  

---

## Public Subnet

The public subnet contains resources exposed to the internet.

Components:

- Application Load Balancer
- NAT Gateway

The Internet Gateway provides internet access.

---

## Private Application Subnet

Private application subnets host the compute layer.

Components:

- ECS services
- ECS tasks
- EC2 instances (PROD)

Outbound traffic is routed through the NAT Gateway.

---

## Private Database Subnet

Database resources are deployed inside private database subnets.

Components:

- Amazon RDS PostgreSQL

These subnets are not internet accessible.

---

# 6. NAT Gateway

The NAT Gateway allows resources in private subnets to access the internet securely.

Use cases:

- Pull container images from ECR
- External API communication
- System updates

Traffic flow:

Private Subnet → NAT Gateway → Internet

---

# 7. Compute Layer

## Amazon Elastic Container Service (ECS)

The platform uses ECS to orchestrate containerized services.

Two compute models are used.

---

## DEV Environment – ECS Fargate

The development environment uses **ECS Fargate**.

Advantages:

- No EC2 management
- Simplified infrastructure
- Fast deployments

Components:

- ECS Cluster
- ECS Service
- Task Definitions
- Fargate Tasks
- Auto Scaling

---

## PROD Environment – ECS on EC2

Production workloads run on ECS using EC2 instances.

Advantages:

- Greater resource control
- Cost optimization
- Predictable performance

Infrastructure:

- ECS Cluster
- ECS Service
- ECS Tasks
- Capacity Provider
- Auto Scaling Group
- Launch Template
- EC2 Container Instances

---

# 8. Data Layer

## Amazon RDS PostgreSQL

Amazon RDS provides relational database services.

### Development Environment

Single instance deployment used for development and testing.

### Production Environment

Production uses **Multi-AZ deployment**.

Components:

- Primary database instance
- Standby instance

Benefits:

- High availability
- Automatic failover
- Data redundancy

---

# 9. Application Storage

## Amazon S3 – Application Storage

S3 stores application-generated files.

Examples:

- Uploaded documents
- Profile images
- Attachments
- Reports

Backend services interact directly with this storage.

---

# 10. Container Registry

## Amazon Elastic Container Registry (ECR)

ECR stores container images used by the application.

Responsibilities:

- Docker image storage
- Image versioning
- Integration with CI/CD
- Secure access by ECS tasks

Deployment flow:

CI/CD → ECR → ECS

---

# 11. CI/CD Pipeline

## GitHub

GitHub hosts the source code repositories for:

- Frontend
- Backend
- Infrastructure documentation

---

## Continuous Integration / Continuous Deployment

The CI/CD pipeline automates build and deployment.

Pipeline responsibilities:

- Build frontend artifacts
- Build Docker images
- Run tests
- Push images to ECR
- Deploy frontend to S3
- Deploy backend to ECS

Deployment flow:

Developer → GitHub → CI/CD → ECR → ECS  
CI/CD → S3 Frontend

---

# 12. Security Architecture

Security is implemented using multiple AWS services.

## AWS IAM

Identity and Access Management controls permissions for:

- ECS tasks
- EC2 instances
- CI/CD pipelines
- Engineers

---

## AWS Secrets Manager

Secrets Manager securely stores sensitive configuration values.

Examples:

- Database credentials
- API keys
- Authentication tokens

ECS services retrieve secrets at runtime.

---

## Security Groups

Security groups control network access between components.

Examples:

ALB → ECS  
ECS → RDS  

---

# 13. Observability

Operational monitoring is provided through Amazon CloudWatch.

## CloudWatch Logs

Centralized log collection from:

- ECS containers
- EC2 instances
- Application services

---

## CloudWatch Metrics

Infrastructure monitoring includes:

- ECS CPU and memory utilization
- EC2 metrics
- ALB request metrics
- RDS performance metrics

---

## CloudWatch Alarms

Alerts are configured for abnormal conditions such as:

- High CPU usage
- Application failures
- Infrastructure degradation

---

# 14. Cost Management

## AWS Budgets

Budget alerts notify engineers when spending thresholds are reached.

---

## AWS Cost Explorer

Provides visibility into AWS cost breakdowns by service and environment.

---

# 15. Backup Strategy

## AWS Backup

AWS Backup provides centralized backup management.

Components:

- Backup Plan
- Backup Vault
- Scheduled Backups

Backups are configured for:

RDS DEV  
RDS PROD  

This enables recovery from failures or accidental data loss.

---

# 16. Summary

The AID HealthTech architecture provides a modern, scalable, and secure cloud platform.

Key characteristics:

- CDN-based content delivery
- Containerized compute with ECS
- Environment isolation
- High availability database architecture
- Secure secret management
- Centralized monitoring
- Automated backup strategy
- Controlled infrastructure costs

This architecture allows the platform to scale while maintaining operational stability and security.
