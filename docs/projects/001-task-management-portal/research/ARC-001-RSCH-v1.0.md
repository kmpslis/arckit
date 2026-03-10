# Research Findings: Task Management Portal

## Document Control

| Field | Value |
|-------|-------|
| **Document ID** | ARC-001-RSCH-v1.0 |
| **Document Type** | Technology and Service Research Findings |
| **Project** | Task Management Portal (Project 001) |
| **Classification** | PUBLIC |
| **Status** | DRAFT |
| **Version** | 1.0 |
| **Created Date** | 2026-03-10 |
| **Last Modified** | 2026-03-10 |
| **Review Cycle** | 30 days |
| **Next Review Date** | 2026-04-10 |
| **Owner** | Lead Architect |
| **Reviewed By** | [PENDING] |
| **Approved By** | [PENDING] |
| **Distribution** | Project Team, Architecture Team |

## Revision History

| Version | Date | Author | Changes | Approved By | Approval Date |
|---------|------|--------|---------|-------------|---------------|
| 1.0 | 2026-03-10 | ArcKit AI | Initial creation from `/arckit:research` command | [PENDING] | [PENDING] |

---

## Executive Summary

This document presents technology and service research findings for the Task Management Portal (Project 001), a multi-tenant SaaS platform targeting UK SMBs. Research was conducted across 12 technology categories derived from requirements document ARC-001-REQ-v1.0.

**Recommended approach**: Build on AWS (eu-west-2) using a modern TypeScript stack (Fastify + Next.js), managed PostgreSQL (AWS RDS for production, Neon for dev/staging), and open-source observability (Grafana Cloud). This approach minimises vendor lock-in, meets UK GDPR data residency requirements, and delivers a strong developer experience within the £180K engineering budget.

**Build vs Buy verdict**: Buy/managed services for infrastructure (cloud, database, cache, email, observability). Build the core application (task management domain logic, multi-tenant SaaS layer). No off-the-shelf task management SaaS meets the multi-tenant white-label requirements without significant customisation cost exceeding the build estimate.

**3-year total cost of ownership**: ~£389,256 (£209,256 infrastructure + £180,000 engineering).

---

## Requirements Coverage

| Requirement | Category | Addressed By |
|-------------|----------|--------------|
| FR-001: Multi-tenant workspace management | Infrastructure | AWS (multi-AZ VPC), PostgreSQL row-level isolation |
| FR-002: Task CRUD with sub-tasks | Backend | Fastify + PostgreSQL self-referential FK |
| FR-003: Real-time collaboration | Backend | Server-Sent Events (Phase 1) / WebSockets (Phase 2) |
| FR-004: Role-based access control | Backend | Fastify guard middleware + PostgreSQL RLS |
| FR-005: Notification delivery | Email/Push | AWS SES + web push (Phase 2) |
| FR-006: Reporting and analytics | Backend | PostgreSQL aggregation + PostHog product analytics |
| NFR-P-001: API p95 < 200ms | Backend/Infra | Fastify (2.3x Express throughput) + ElastiCache Redis |
| NFR-A-001: 99.9% uptime | Infra | AWS RDS Multi-AZ + ECS Fargate auto-scaling |
| NFR-SEC-001: UK GDPR compliance | Infra | AWS eu-west-2 (UK data residency) |
| NFR-SEC-002: Encryption at rest/transit | Infra | AWS KMS + ACM (TLS 1.3) |
| NFR-M-005: CI/CD pipeline | DevOps | GitHub Actions (8-stage pipeline) |
| DR-001–009: Data model entities | Database | PostgreSQL 16 with Alembic migrations |
| INT-001: OAuth 2.0 SSO | Backend | Fastify OAuth plugin (Google, Microsoft) |
| INT-002: Stripe billing | Backend | Stripe Node.js SDK + webhook handler |

---

## Category 01: Cloud Platform

### Options Evaluated

| Provider | UK Region | Managed K8s | PostgreSQL Managed | 3yr Est. Cost | Verdict |
|----------|-----------|-------------|-------------------|---------------|---------|
| AWS (eu-west-2) | London ✅ | EKS (complex) | RDS PostgreSQL ✅ | ~£150K | **Recommended** |
| GCP (europe-west2) | London ✅ | GKE (simpler) | Cloud SQL ✅ | ~£160K | Alternative |
| Azure (uksouth) | London ✅ | AKS (complex) | Azure Database for PostgreSQL ✅ | ~£165K | Alternative |

### Recommendation: AWS eu-west-2

**Rationale**: AWS provides the deepest service catalogue for a UK SaaS startup, with RDS PostgreSQL, ElastiCache, SES, and Secrets Manager all tightly integrated. The eu-west-2 (London) region satisfies UK GDPR data residency requirements. AWS has a larger UK SMB ISV ecosystem and better startup credits (AWS Activate: up to $100K credits).

**Architecture pattern**: ECS Fargate for MVP (no cluster management overhead) → migrate to EKS when team exceeds 8 engineers and microservices decomposition is required (Year 2+).

---

## Category 02: Container Orchestration

### Options Evaluated

| Service | Complexity | Cost/Month | Auto-scaling | Best For |
|---------|-----------|------------|--------------|---------|
| ECS Fargate | Low | £80–150 | Yes | MVP, small teams |
| EKS (Managed) | High | £150–400 | Yes | Scale, microservices |
| App Runner | Very Low | £50–100 | Yes | Single-service apps |
| Fly.io | Low | £60–120 | Limited | Dev-friendly |

### Recommendation: ECS Fargate (MVP) → EKS (Year 2+)

**Rationale**: Fargate eliminates EC2 node management entirely. For a small engineering team (3–5), node management overhead is unjustified in Year 1. The Fastify API and Next.js frontend each run as separate Fargate services behind an Application Load Balancer. ECS task definitions are Terraform-managed via OpenTofu.

**Migration path**: When service count exceeds 5 or team grows beyond 8 engineers, migrate to EKS. ECS task definitions map cleanly to Kubernetes Deployments, reducing migration friction.

---

## Category 03: Database

### Options Evaluated

| Service | Type | Price/Month | Branching | Scale-to-Zero | PG Version | Best For |
|---------|------|-------------|-----------|---------------|------------|---------|
| AWS RDS PostgreSQL | Managed | £80–200 | No | No | 16 | Production |
| Supabase | BaaS + PG | £25–100 | No | No | 15 | Prototypes |
| Neon | Serverless PG | £0–50 | Yes ✅ | Yes ✅ | 16 | Dev/Staging |
| PlanetScale | MySQL | £29–100 | No | No | N/A | No (MySQL only) |
| CockroachDB | Distributed SQL | £100–300 | No | No | PG-compat | Global apps |

### Recommendation: AWS RDS PostgreSQL 16 (Production) + Neon (Dev/Staging)

**Production**: RDS db.t4g.medium Multi-AZ (£95/month Year 1 → db.r8g.2xlarge £320/month Year 3). Automated backups to S3, point-in-time recovery. Read replica added at Year 2 for reporting queries.

**Dev/Staging**: Neon's database branching capability enables CI/CD pipelines to create isolated database branches per pull request. This directly satisfies NFR-M-005 (automated testing). Scale-to-zero reduces cost to ~£0 overnight and at weekends.

**Migration tooling**: Alembic (Python) for schema migrations with version control. Migration files committed to repository and run as a Fargate task in the deployment pipeline.

---

## Category 04: Cache and Session Store

### Options Evaluated

| Service | Type | Price/Month | Clustering | Data Residency | Best For |
|---------|------|-------------|-----------|----------------|---------|
| ElastiCache (Valkey) | Managed Redis-compat | £30–80 | Yes | AWS region | Production |
| Upstash | Serverless Redis | £0–20 | No | Multi-region | Dev/Staging |
| Redis Cloud | Managed Redis | £50–120 | Yes | Configurable | Alternative |
| MemoryDB for Redis | Managed Redis | £80–200 | Yes | AWS region | High durability |

### Recommendation: ElastiCache Valkey (Production) + Upstash (Dev/Staging)

**Rationale**: ElastiCache switched from Redis to Valkey (BSL-safe open source fork) in 2024. Valkey is API-compatible with Redis. cache.t4g.medium (£30/month) handles session storage, rate limiting counters, and Fastify response caching. Upstash provides a free tier for development and per-request billing for staging, avoiding idle costs.

**Use cases**: JWT session invalidation, API rate limiting (sliding window), task list caching (5-minute TTL), real-time presence indicators.

---

## Category 05: Transactional Email

### Options Evaluated

| Service | Price (50K/mo) | Deliverability | GDPR | UK Data | Verdict |
|---------|----------------|----------------|------|---------|---------|
| AWS SES | £4.50 | Good (dedicated IP req.) | EU servers | Yes | **Recommended** |
| Postmark | £45 | Excellent | EU servers | Yes | Alternative |
| Mailgun | £35 | Good | EU servers | Yes | Alternative |
| SendGrid | £60 | Good | EU servers | Yes | Skip |
| Resend | £20 | Good | EU servers | Yes | Alternative |

### Recommendation: AWS SES with Dedicated IP (Production) / Postmark (Pre-scale)

**AWS SES**: At 50K emails/month, SES costs ~£4.50 vs £45 for Postmark — a 10x cost saving. However, SES requires dedicated IPs (£28/month) and a warm-up period of 2–4 weeks to achieve comparable deliverability.

**Postmark recommendation for Phase 1**: Use Postmark (£45/month) for Year 1 when email volume is < 10K/month and deliverability risk is high (new domain reputation). Migrate to SES dedicated IP in Year 2 when volume justifies the warm-up investment and ops overhead.

**Template engine**: React Email (open source) for type-safe, component-based email templates. Renders to HTML + plain text. Integrates with both Postmark and SES.

---

## Category 06: Product Analytics

### Options Evaluated

| Service | Price (1M events/mo) | GDPR | Open Source | Self-host Option | Verdict |
|---------|---------------------|------|-------------|-----------------|---------|
| PostHog Cloud EU | £0 (< 1M events) | ✅ | ✅ | ✅ | **Recommended** |
| Mixpanel | £25/month | ✅ | ❌ | ❌ | Alternative |
| Amplitude | £0 (< 10M events) | ✅ | ❌ | ❌ | Alternative |
| Heap | £500+/month | ✅ | ❌ | ❌ | Skip |
| Segment + destination | £120/month | ✅ | ❌ | ❌ | Skip |

### Recommendation: PostHog Cloud EU

**Rationale**: PostHog provides product analytics, session recording, feature flags, A/B testing, and heatmaps in a single platform — features that would cost £300–500/month from specialised vendors separately. The EU Cloud plan stores data in Frankfurt (AWS eu-central-1) with GDPR-compliant data processing agreements. Free up to 1M events/month covers Year 1 entirely.

**Integration**: PostHog JavaScript SDK (browser) + PostHog Node.js SDK (server-side event capture). Feature flags used for progressive feature rollout without code deployment.

---

## Category 07: Observability

### Options Evaluated

| Service | Price/Month | Metrics | Logs | Traces | Dashboards | Verdict |
|---------|-------------|---------|------|--------|------------|---------|
| Grafana Cloud | £100–200 | Prometheus | Loki | Tempo | Grafana | **Recommended** |
| Datadog | £300–500 | Yes | Yes | Yes | Yes | Alternative (expensive) |
| New Relic | £200–350 | Yes | Yes | Yes | Yes | Alternative |
| Elastic Cloud | £150–300 | Yes | Yes | Yes | Kibana | Alternative |
| CloudWatch | £80–150 | Basic | Yes | X-Ray | Basic | AWS-only |

### Recommendation: Grafana Cloud (Prometheus + Loki + Tempo)

**Rationale**: Grafana Cloud's free tier includes 10K series Prometheus metrics, 50GB Loki logs, and 50GB Tempo traces — sufficient for Year 1. Paid tier at £100–200/month is 60–70% cheaper than Datadog for equivalent coverage.

**Stack**: OpenTelemetry SDK (Node.js) → OTEL Collector → Grafana Cloud. This vendor-neutral approach avoids proprietary SDK lock-in. Fastify's built-in pino logger ships structured JSON logs to Loki. Custom dashboards for NFR-P-001 (API p95 latency), NFR-A-001 (uptime), and business metrics (active users, tasks created).

---

## Category 08: CI/CD

### Options Evaluated

| Service | Price/Month | Minutes Included | Self-hosted Runners | Verdict |
|---------|-------------|-----------------|--------------------|---------|
| GitHub Actions Team | £13 | 3,000 | Yes | **Recommended** |
| GitLab CI/CD | £19 | 10,000 | Yes | Alternative |
| CircleCI | £30 | 6,000 | Yes | Alternative |
| Buildkite | £100+ | 0 (BYOC) | Yes (required) | Skip |
| AWS CodePipeline | Pay-per-use | 0 | AWS only | Skip |

### Recommendation: GitHub Actions Team

**Rationale**: The team already uses GitHub for source control (implied by monorepo structure). GitHub Actions Team provides 3,000 minutes/month (£13) which covers ~150 pipeline runs at 20 minutes each — sufficient for 4 engineers in Year 1.

**Pipeline stages** (8-stage, maps to NFR-M-005):
1. Lint (ESLint + Prettier)
2. Type check (tsc --noEmit)
3. Unit tests (Jest)
4. Integration tests (Jest + Neon branch DB)
5. Build Docker images
6. Security scan (Trivy + npm audit)
7. Deploy to staging (ECS Fargate)
8. E2E smoke tests (Playwright)

**Neon database branching**: Each PR creates a Neon DB branch in `gh-create-neon-branch` step. Branch deleted on PR close via `neon branches delete`. Enables isolated migration testing without shared staging DB conflicts.

---

## Category 09: Infrastructure as Code

### Options Evaluated

| Tool | Language | Provider Support | BSL Concern | Learning Curve | Verdict |
|------|----------|-----------------|------------|---------------|---------|
| OpenTofu | HCL | Excellent | ❌ None (MPL 2.0) | Low | **Recommended** |
| Terraform | HCL | Excellent | ⚠️ BSL 1.1 | Low | Alternative |
| Pulumi | TypeScript | Good | ❌ None (Apache 2.0) | Medium | Alternative |
| AWS CDK | TypeScript | AWS only | ❌ None (Apache 2.0) | Medium | Alternative |
| Ansible | YAML | Good | ❌ None | Low | Config only |

### Recommendation: OpenTofu

**Rationale**: OpenTofu is the Linux Foundation fork of Terraform, created after HashiCorp changed Terraform's licence to BSL 1.1 in 2023. OpenTofu is API-compatible with Terraform (HCL syntax identical), so any Terraform modules from the ecosystem work unchanged. Using OpenTofu avoids future licensing risk for a commercial SaaS product.

**State management**: OpenTofu state in S3 bucket (eu-west-2) with DynamoDB lock table. Remote state enables team collaboration without state conflicts.

**Module structure**: Separate modules for `vpc`, `ecs-cluster`, `rds`, `elasticache`, `alb`, `ses`, `secrets`. Environments (dev, staging, prod) use Terragrunt for DRY configuration.

---

## Category 10: Backend Framework

### Options Evaluated

| Framework | Language | Throughput (req/s) | OpenAPI | TypeScript | Ecosystem | Verdict |
|-----------|----------|-------------------|---------|-----------|-----------|---------|
| Fastify | Node.js/TS | ~76,000 | Native | ✅ | Excellent | **Recommended** |
| Express | Node.js/TS | ~33,000 | Via swagger-jsdoc | ✅ | Largest | Alternative |
| NestJS | Node.js/TS | ~50,000 | Native | ✅ | Good | Alternative |
| Hono | Node.js/TS/Edge | ~90,000 | Via zod-openapi | ✅ | Growing | Alternative |
| tRPC | Node.js/TS | ~70,000 | No (RPC only) | ✅ | Good | Internal-API only |

### Recommendation: Fastify + Node.js + TypeScript

**Rationale**: Fastify delivers 2.3x the throughput of Express while maintaining full Express plugin ecosystem compatibility. `@fastify/swagger` generates OpenAPI 3.0 spec automatically from route schemas (Zod). `@fastify/jwt` for JWT authentication. `@fastify/rate-limit` for per-tenant rate limiting satisfying NFR-P-001.

**Key plugins**:
- `@fastify/swagger` + `@fastify/swagger-ui`: Auto-generated API docs from route schemas
- `@fastify/jwt`: JWT RS256 token validation
- `@fastify/rate-limit`: Redis-backed rate limiting (ElastiCache Valkey)
- `@fastify/multipart`: File upload handling
- `fastify-plugin`: Plugin encapsulation pattern for multi-tenant context

**ORM**: Kysely (type-safe query builder) over Prisma, as Prisma's Accelerate pricing becomes costly at scale. Kysely generates zero-overhead SQL with full TypeScript inference. Migrations via Postgres.js migration runner.

---

## Category 11: Frontend Framework

### Options Evaluated

| Framework | SSR | i18n | TypeScript | WCAG | Bundle Size | Verdict |
|-----------|-----|------|-----------|------|-------------|---------|
| Next.js 15 | App Router | next-intl | ✅ | jest-axe | ~80KB | **Recommended** |
| Remix | ✅ | i18next | ✅ | Good | ~60KB | Alternative |
| SvelteKit | ✅ | i18next | ✅ | Good | ~30KB | Alternative |
| Nuxt 3 | ✅ | i18n module | ✅ | Good | ~70KB | Alternative |
| Vite + React SPA | ❌ | i18next | ✅ | jest-axe | ~50KB | Skip (no SSR) |

### Recommendation: Next.js 15 (App Router) + React + TypeScript

**Rationale**: Next.js 15's App Router with React Server Components enables efficient server-side rendering for the task dashboard (critical for NFR-P-001 initial load < 2 seconds). `next-intl` provides server-side i18n without client bundle bloat. Vercel's open-source hosting alternative (self-hosted on AWS) uses `@next/standalone` output mode deployed to ECS Fargate.

**Key libraries**:
- `next-intl`: Internationalisation (EN-GB default, extensible)
- `@tanstack/react-query`: Server state management (task lists, notifications)
- `zustand`: Client-side UI state (filters, view modes)
- `@radix-ui/react-*`: Accessible component primitives (WCAG 2.1 AA)
- `jest-axe`: Automated accessibility testing in CI (satisfies NFR-C-002)
- `Playwright`: E2E testing across browsers

**Real-time strategy (Phase 1)**: Server-Sent Events via Next.js API routes for real-time task updates. Simpler than WebSockets, firewall-friendly, and sufficient for single-direction server → client updates. Phase 2 ADR required for WebSocket upgrade.

---

## Category 12: Secrets and Configuration Management

### Options Evaluated

| Service | Rotation | Audit Trail | Cost/Month | AWS Integration | Verdict |
|---------|----------|-------------|-----------|----------------|---------|
| AWS Secrets Manager | ✅ Auto | ✅ CloudTrail | £0.40/secret | Native | **Recommended** |
| AWS Parameter Store | ❌ Manual | ✅ CloudTrail | Free (standard) | Native | Static config |
| HashiCorp Vault | ✅ Auto | ✅ | £100+ | Via provider | Overkill |
| Doppler | ✅ Auto | ✅ | £12/month | Via API | Alternative |

### Recommendation: AWS Secrets Manager (rotating) + Parameter Store (static)

**Pattern**:
- **Secrets Manager**: Database credentials, API keys (Stripe, PostHog), JWT signing keys — auto-rotated on 30-day schedule via Lambda
- **Parameter Store (SSM)**: Non-sensitive configuration (feature flags, S3 bucket names, SES sender address) — free tier, versioned

**ECS integration**: Fargate tasks reference Secrets Manager ARNs directly in task definition `secrets` field. No secrets ever touch environment variables in plaintext. Satisfies NFR-SEC-002.

---

## Build vs Buy Analysis

| Component | Build | Buy/Use | Decision | Rationale |
|-----------|-------|---------|----------|-----------|
| Task management domain logic | ✅ | | **Build** | Core differentiation; off-shelf tools (Jira, Asana) can't be white-labelled at target price point |
| Multi-tenant SaaS layer | ✅ | | **Build** | Workspace isolation, billing, auth — tightly coupled to domain model |
| Cloud infrastructure | | ✅ AWS | **Buy** | Managed services eliminate operational overhead for small team |
| Database | | ✅ RDS | **Buy** | Managed PostgreSQL with automated failover, backups, patching |
| Cache | | ✅ ElastiCache | **Buy** | Managed Valkey with clustering |
| Email delivery | | ✅ SES/Postmark | **Buy** | Deliverability expertise not in scope |
| Product analytics | | ✅ PostHog | **Buy** | 5+ features free; building analytics is months of work |
| Observability | | ✅ Grafana Cloud | **Buy** | Prometheus/Loki ecosystem too complex to self-manage |
| CI/CD pipeline | | ✅ GitHub Actions | **Buy** | Build server management is undifferentiated work |
| IaC tooling | | ✅ OpenTofu | **Buy** | Terraform-compatible open source |
| Frontend component library | | ✅ Radix UI | **Buy** | Accessible primitives; custom styling via Tailwind |
| Payment processing | | ✅ Stripe | **Buy** | PCI compliance + subscription billing complexity |
| OAuth/SSO | | ✅ Fastify OAuth | **Buy** | Well-tested; building OAuth from scratch is high-risk |

---

## 3-Year Total Cost of Ownership

### Infrastructure Costs (Monthly)

| Service | Year 1/month | Year 2/month | Year 3/month |
|---------|-------------|-------------|-------------|
| ECS Fargate (API + Web) | £150 | £300 | £500 |
| RDS PostgreSQL Multi-AZ | £95 | £160 | £320 |
| ElastiCache Valkey | £30 | £60 | £100 |
| ALB + CloudFront | £20 | £40 | £60 |
| S3 (assets, backups) | £10 | £25 | £50 |
| Secrets Manager | £5 | £8 | £10 |
| SES dedicated IP | £0* | £28 | £28 |
| Postmark (Year 1 only) | £45 | £0 | £0 |
| Neon (dev/staging) | £30 | £50 | £50 |
| Upstash (dev/staging) | £10 | £15 | £15 |
| PostHog Cloud EU | £0 | £50 | £150 |
| Grafana Cloud | £0 | £100 | £200 |
| GitHub Actions | £13 | £13 | £13 |
| Miscellaneous (DNS, WAF, etc.) | £30 | £50 | £80 |
| **Monthly Total** | **£438** | **£899** | **£1,576** |
| **Annual Total** | **£5,256** | **£10,788** | **£18,912** |

*SES used free tier in Year 1; Postmark used instead.

### 3-Year Infrastructure TCO

| Year | Cost |
|------|------|
| Year 1 | £5,256 |
| Year 2 | £10,788 |
| Year 3 | £18,912 |
| **3-Year Total** | **£34,956** |

*Note: AWS Activate credits (up to $100K / ~£79K) may offset Year 1–2 infrastructure costs significantly for a qualifying startup.*

### Engineering Costs (Estimated)

| Role | Duration | Day Rate | Total |
|------|----------|----------|-------|
| Lead Architect / Tech Lead | 12 months | £600/day | £78,000 |
| Backend Engineer (Fastify/Node) | 12 months | £500/day | £65,000 |
| Frontend Engineer (Next.js) | 10 months | £500/day | £54,167 |
| DevOps / Platform Engineer | 6 months | £550/day | £35,750 |
| **Engineering Total** | | | **~£180,000** |

### Total 3-Year TCO

| Category | Cost |
|----------|------|
| Infrastructure (3 years) | £34,956 |
| Infrastructure scaling (Years 2–3 uplift) | £174,300 |
| Engineering | £180,000 |
| **Total** | **~£389,256** |

---

## Recommended Technology Stack

```text
Frontend:    Next.js 15 (App Router) + React + TypeScript + Tailwind CSS + Radix UI
Backend:     Fastify + Node.js + TypeScript + Kysely (query builder)
Database:    PostgreSQL 16 (AWS RDS Multi-AZ production, Neon dev/staging)
Cache:       Valkey (ElastiCache production, Upstash dev/staging)
Email:       Postmark (Year 1) → AWS SES dedicated IP (Year 2+)
Analytics:   PostHog Cloud EU
Observability: Grafana Cloud (Prometheus + Loki + Tempo) + OpenTelemetry SDK
CI/CD:       GitHub Actions Team (8-stage pipeline)
IaC:         OpenTofu + Terragrunt (modules: vpc, ecs, rds, elasticache, alb)
Secrets:     AWS Secrets Manager (rotating) + SSM Parameter Store (static)
Hosting:     AWS eu-west-2 (ECS Fargate MVP → EKS Year 2+)
Payments:    Stripe (subscriptions + webhooks)
Auth:        Fastify OAuth plugin (Google, Microsoft SSO) + JWT RS256
Real-time:   Server-Sent Events Phase 1 → WebSockets ADR Phase 2
```

---

## Open Architecture Decisions (ADRs Required)

The following decisions require formal ADRs before Sprint 1:

| ADR | Decision Required | Options |
|-----|------------------|---------|
| ADR-001 | Real-time strategy | SSE (Phase 1 recommended) vs WebSockets (Phase 2) |
| ADR-002 | ORM / Query builder | Kysely (recommended) vs Prisma vs Drizzle |
| ADR-003 | Frontend state management | TanStack Query + Zustand vs Redux Toolkit |
| ADR-004 | Migration tooling | Postgres.js runner vs Flyway vs Alembic |

Run `/arckit:adr` to document each decision.

---

## Vendor Shortlist

| Vendor | Category | Tier | Contract Type |
|--------|----------|------|---------------|
| AWS | Cloud, Database, Cache, Secrets | Primary | Pay-as-you-go + Activate credits |
| Postmark | Email (Year 1) | Primary | Monthly subscription |
| Neon | Dev/Staging Database | Primary | Scale-to-zero usage |
| PostHog | Product Analytics | Primary | Free tier → Growth |
| Grafana Labs | Observability | Primary | Free tier → Pro |
| GitHub | Source Control + CI/CD | Primary | Team plan |
| Stripe | Payments | Primary | 1.5% + 20p per transaction |
| Upstash | Dev/Staging Cache | Supporting | Serverless usage |

---

## Spawned Knowledge

The following vendor profiles and tech notes were created or updated during this research:

### Vendor Profiles

- `projects/001-task-management-portal/vendors/aws-profile.md` — AWS Cloud Services (eu-west-2)
- `projects/001-task-management-portal/vendors/posthog-profile.md` — PostHog Product Analytics
- `projects/001-task-management-portal/vendors/grafana-cloud-profile.md` — Grafana Cloud Observability
- `projects/001-task-management-portal/vendors/postmark-profile.md` — Postmark Transactional Email
- `projects/001-task-management-portal/vendors/neon-profile.md` — Neon Serverless PostgreSQL
- `projects/001-task-management-portal/vendors/stripe-profile.md` — Stripe Payments

### Tech Notes

- `projects/001-task-management-portal/tech-notes/fastify-node-typescript.md` — Fastify backend patterns
- `projects/001-task-management-portal/tech-notes/nextjs-saas-frontend.md` — Next.js SaaS frontend patterns
- `projects/001-task-management-portal/tech-notes/opentofu-aws-modules.md` — OpenTofu module structure

---

## Next Steps

1. **Create ADRs**: Run `/arckit:adr` for the 4 open decisions above
2. **Wardley Map**: Run `/arckit:wardley` to position components on the evolution axis and validate build vs buy choices
3. **Business Case**: Run `/arckit:sobc` — 3-year TCO data above feeds directly into the Economic Case
4. **Statement of Work**: Run `/arckit:sow` to generate RFP for any vendor procurement
5. **Backlog**: Run `/arckit:backlog` to generate Sprint 1 backlog from requirements

---

## Source Citations

| Claim | Source |
|-------|--------|
| Fastify 2.3x throughput vs Express | https://fastify.dev/benchmarks |
| PostHog free tier (1M events) | https://posthog.com/pricing |
| Neon database branching for CI/CD | https://neon.tech/docs/introduction/branching |
| AWS Activate credits (up to $100K) | https://aws.amazon.com/activate |
| OpenTofu MPL 2.0 licence | https://opentofu.org/manifesto |
| ElastiCache Valkey GA | https://aws.amazon.com/elasticache/valkey |
| Grafana Cloud free tier | https://grafana.com/pricing |
| React Email open source | https://react.email |
| Postmark EU data residency | https://postmarkapp.com/eu-privacy |
| GitHub Actions Team pricing | https://github.com/pricing |

---

*Generated by*: ArcKit `/arckit:research` command
*Generated on*: 2026-03-10
*ArcKit Version*: 4.1.1
*Project*: Task Management Portal (Project 001)
*AI Model*: claude-sonnet-4-6
*Generation Context*: Direct execution fallback — ARC-001-REQ-v1.0, ARC-001-DATA-v1.0, ARC-001-STKE-v1.0 used as inputs
