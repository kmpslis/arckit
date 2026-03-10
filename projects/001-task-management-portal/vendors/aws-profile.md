# Vendor Profile: Amazon Web Services (AWS)

**Slug**: aws
**Last Researched**: 2026-03-10
**Confidence**: High (5+ data points)

## Overview

Amazon Web Services (AWS) is the world's leading cloud provider by market share (~35% globally, ~35% UK). AWS offers 200+ managed services across compute, storage, database, networking, security, AI/ML, and developer tooling. AWS was the first major cloud provider (2006) and retains the broadest service catalogue and most mature managed service ecosystem.

**UK Presence**: AWS has operated the eu-west-2 (London) region since 2016. The region comprises three availability zones (eu-west-2a, eu-west-2b, eu-west-2c), making it suitable for multi-AZ production deployments. A second UK region (eu-west-4, Manchester) was in development as of 2025.

**Strategic Position**: Market leader. Used by a majority of UK SaaS startups. Dominant in financial services, NHS Digital, and UK government (through G-Cloud).

## Products and Services

| Service | Category | Notes |
|---------|----------|-------|
| EC2 | Compute | Virtual machines; multiple instance families |
| ECS / Fargate | Container Orchestration | Managed containers; Fargate is serverless |
| EKS | Kubernetes | Managed Kubernetes control plane |
| RDS | Managed Database | PostgreSQL, MySQL, Oracle, SQL Server, MariaDB |
| Aurora | Database | PostgreSQL/MySQL-compatible; higher throughput |
| ElastiCache | Caching | Redis and Memcached managed |
| S3 | Object Storage | Highly durable; 11-nines durability |
| CloudFront | CDN | Global edge; 450+ PoPs |
| ALB | Load Balancer | Application Layer 7 load balancing |
| SES | Email Delivery | Transactional and marketing email |
| Secrets Manager | Secrets | Managed secrets with automated rotation |
| KMS | Key Management | Encryption key management; HSM-backed |
| IAM | Identity | Fine-grained access control |
| CloudTrail | Audit Logging | API call logging; immutable |
| CloudWatch | Monitoring | Metrics, logs, alarms |
| X-Ray | Distributed Tracing | APM tracing |
| VPC | Networking | Private virtual network |

## Pricing Model

- Pay-as-you-go for most services; no upfront commitment required
- Reserved Instances (1 or 3 year) for 30–60% savings on EC2 and RDS
- Savings Plans (compute) offer flexibility across instance types
- AWS Activate programme: $25K–$100K credits for eligible startups (apply at activate.aws)
- Data egress charges apply (first 100GB/month free; ~£0.07/GB thereafter from eu-west-2)

**Key pricing data points (eu-west-2, as of mid-2025)**:
- ECS Fargate: ~£0.04 vCPU/hour + £0.004/GB RAM/hour
- RDS db.t3.medium Multi-AZ: ~£95/month (on-demand)
- ElastiCache cache.t3.micro: ~£30/month
- S3 Standard: ~£0.023/GB/month
- CloudFront: ~£0.0085/GB egress (eu-west-2)
- Secrets Manager: £0.40/secret/month + £0.05/10K API calls

## UK Government Presence

- AWS is listed on G-Cloud 14 (Digital Marketplace)
- AWS GovCloud equivalent for UK is not required for this project (private sector SaaS)
- AWS eu-west-2 (London) provides UK GDPR-compliant data residency
- AWS has SCCs (Standard Contractual Clauses) in place for EU data transfers

## Strengths

- Broadest service catalogue of any cloud provider
- Most mature Terraform provider (`hashicorp/aws`) — largest community, most examples
- Dominant UK hiring pool for cloud/DevOps roles
- AWS Activate credits dramatically reduce Year 1 costs for startups
- Native integrations across all services within the same VPC (no egress between services)
- eu-west-2 has full feature parity with us-east-1 for all services in this stack
- Comprehensive audit trail via CloudTrail (satisfies NFR-C-002)

## Weaknesses

- Pricing complexity — AWS bills are notoriously difficult to predict and optimise
- Data egress costs can be significant at high traffic volumes
- Some services (EKS control plane at ~£65/month) have fixed costs regardless of usage
- Console UX can be overwhelming; steep learning curve for new engineers
- Vendor lock-in risk at service level (SES, ElastiCache, Secrets Manager)

## Projects Referenced In

- Task Management Portal (Project 001) — Primary cloud provider recommendation

## External Links

- Pricing: https://aws.amazon.com/pricing/
- AWS Activate: https://aws.amazon.com/activate/
- Terraform Provider: https://registry.terraform.io/providers/hashicorp/aws/latest
