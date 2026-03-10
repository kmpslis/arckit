# Vendor Profile: Terraform (HashiCorp)

**Slug**: terraform
**Last Researched**: 2026-03-10
**Confidence**: High (5+ data points)

## Overview

Terraform is a declarative Infrastructure-as-Code tool created by HashiCorp (founded 2012). Written in Go and using HCL (HashiCorp Configuration Language), Terraform enables infrastructure to be defined as code, version-controlled, planned, and applied reproducibly. It supports 3,000+ providers via the Terraform Registry. As of 2023, HashiCorp changed the licence from MPL-2.0 to BSL (Business Source Licence), prompting the OpenTofu fork.

**Created by**: HashiCorp (acquired by IBM in 2024)
**First Release**: 2014
**Language**: HCL (declarative) or JSON
**GitHub Stars (hashicorp/terraform)**: ~43,000
**License**: BSL 1.1 (as of 2023); OpenTofu fork is MPL-2.0

## Products and Services

| Product | Description |
|---------|-------------|
| Terraform CLI | Open-source IaC tool (core) |
| Terraform Cloud | Managed state, plans, and applies (SaaS) |
| Terraform Enterprise | Self-hosted with SSO, audit, Sentinel policies |
| Terraform Registry | Public provider and module registry |
| OpenTofu | Open-source BSL-free fork (Linux Foundation) |

## Pricing Model

- Terraform CLI: Free (open source, BSL licence)
- Terraform Cloud Free: 500 managed resources — sufficient for MVP
- Terraform Cloud Plus: ~$20/user/month (teams with remote state, SSO, Sentinel)
- S3 + DynamoDB self-managed backend: < £1/month (just S3 and DynamoDB costs)
- Terraform Enterprise: Contact sales (~£40K+/year)

**Recommended for MVP**: Self-managed S3 backend (< £1/month); no Terraform Cloud needed until team > 5 engineers.

## UK Government Presence

- Terraform is the de facto IaC standard for UK public and private sector cloud
- Used extensively by NHS Digital, HMRC, DVLA, and UK financial services
- No G-Cloud listing (it's a tool, not a service)
- OpenTofu provides BSL-free alternative for public sector organisations

## Strengths

- De facto IaC standard for AWS — largest community, most examples, most Stack Overflow answers
- `hashicorp/aws` provider covers all services in the recommended stack (ECS, RDS, ElastiCache, Secrets Manager, SES, CloudFront, ALB)
- Declarative `plan` / `apply` workflow provides safe, auditable infrastructure changes
- Excellent GitHub Actions integration (`hashicorp/setup-terraform` action)
- Module ecosystem: public registry has production-ready modules for EKS, RDS, VPC
- OpenTofu fork provides BSL-free alternative with identical HCL compatibility
- `tflint` and `checkov` provide IaC linting and security scanning in CI pipelines

## Weaknesses

- BSL licence change in 2023 introduced uncertainty for some organisations
- HCL is not a general-purpose language — complex logic (loops, conditionals) can be awkward
- State management requires care — corrupted state can be disruptive
- `terraform destroy` is irreversible — requires discipline and CI guardrails
- Slower to add new AWS service support compared to AWS CDK (which is AWS-native)
- No native secret management — secrets must be sourced from Secrets Manager via data sources

## Projects Referenced In

- Task Management Portal (Project 001) — IaC tool recommendation (NFR-M-003)

## External Links

- Terraform Registry (AWS provider): https://registry.terraform.io/providers/hashicorp/aws/latest
- OpenTofu: https://opentofu.org
- GitHub Actions integration: https://developer.hashicorp.com/terraform/tutorials/automation/github-actions
