# High-Level Design (HLD) Review: Task Management Portal

> **Template Origin**: Official | **ArcKit Version**: 4.1.1 | **Command**: `/arckit:hld-review`

## Document Control

| Field | Value |
|-------|-------|
| **Document ID** | ARC-001-HLDR-v1.0 |
| **Document Type** | High-Level Design Review |
| **Project** | Task Management Portal (Project 001) |
| **Classification** | PUBLIC |
| **Status** | DRAFT |
| **Version** | 1.1 |
| **Created Date** | 2026-03-11 |
| **Last Modified** | 2026-03-11 |
| **Review Cycle** | On-Demand |
| **Next Review Date** | 2026-04-10 |
| **Owner** | Jane Smith, Head of Engineering |
| **Reviewed By** | [PENDING] |
| **Approved By** | [PENDING] |
| **Distribution** | Project Team, Architecture Team |

## Revision History

| Version | Date | Author | Changes | Approved By | Approval Date |
|---------|------|--------|---------|-------------|---------------|
| 1.0 | 2026-03-11 | ArcKit AI | Initial creation from `/arckit:hld-review` command | [PENDING] | [PENDING] |
| 1.1 | 2026-03-11 | ArcKit AI | BLOCKING-01 resolved — RTO analysis added to DIAG-001; standard failover RTO (< 15 min) separated from PITR data corruption restore (< 4 hr); NFR-A-002 confirmed met | [PENDING] | [PENDING] |

## Document Purpose

This document records the Architecture Review Board's evaluation of the High-Level Design for Quento1's Task Management Portal (Project 001). The HLD under review is the C4 Container Diagram (ARC-001-DIAG-001-v1.0), supplemented by ADR-001 (real-time delivery strategy) and associated architecture decisions documented within the diagram artifact. The review assesses compliance with enterprise architecture principles (ARC-000-PRIN-v1.0) and coverage of requirements (ARC-001-REQ-v1.1) before the project proceeds to detailed design.

---

## 1. Review Overview

### 1.1 Purpose

This document captures the Architecture Review Board's evaluation of the High-Level Design (HLD) for the Task Management Portal. The HLD must demonstrate architectural soundness, alignment with enterprise principles, and feasibility before proceeding to detailed design and implementation sprints.

### 1.2 HLD Document Under Review

| Field | Value |
|-------|-------|
| **Document** | ARC-001-DIAG-001-v1.0 — C4 Container Diagram: Task Management Portal |
| **Supporting Documents** | ARC-001-ADR-001-v1.0 (Real-Time SSE Strategy) |
| **ArcKit Version** | 4.1.1 |
| **Submitted By** | Quento1 Engineering Team (internal) |
| **Submission Date** | 2026-03-10 |

> **Note on HLD Source**: This project does not have a separate vendor HLD document. The C4 Container Diagram (ARC-001-DIAG-001-v1.0) and its embedded architecture decisions, component inventory, security architecture, and NFR coverage tables constitute the HLD artefact under review.

### 1.3 Review Participants

| Name | Role | Organization | Review Focus |
|------|------|--------------|--------------|
| [PENDING] | Lead Reviewer | Enterprise Architecture | Overall architecture, principle compliance |
| [PENDING] | Security Architect | Security | Security architecture, threat model, controls |
| [PENDING] | Domain Architect | Engineering | Domain fit, integration patterns |
| [PENDING] | Infrastructure Architect | Cloud/Infra | Infrastructure, scalability, resilience |
| [PENDING] | Data Architect | Data Governance | Data architecture, privacy, governance |
| [PENDING] | SRE Lead | Operations | Operational readiness, observability |

### 1.4 Review Criteria

- **Architecture Principles**: Compliance with all 16 enterprise principles in ARC-000-PRIN-v1.0
- **Requirements Alignment**: Coverage of all FR/NFR/INT/DR requirements in ARC-001-REQ-v1.1
- **Technical Feasibility**: Soundness and implementability of the proposed design
- **Security and Compliance**: Adequate security controls and GDPR/UK GDPR compliance
- **Scalability and Resilience**: Ability to scale to Year 3 targets and handle failures gracefully
- **Operational Readiness**: Observability, supportability, and deployment automation

---

## 2. Executive Summary

### 2.1 Overall Assessment

**Status**: APPROVED WITH CONDITIONS

The Task Management Portal HLD presents a well-considered, cloud-native architecture for a multi-tenant SaaS product. The design makes strong technology choices — ECS Fargate for stateless compute, PostgreSQL RDS Multi-AZ for persistence, Valkey (ElastiCache) for caching and pub/sub, and Fastify with auto-generated OpenAPI for the API layer. Security controls are notably thorough: JWT RS256 authentication, RBAC at the database role level, AWS Secrets Manager with 30-day rotation, TLS 1.3 everywhere, and an immutable write-once ActivityLog with DB-level REVOKE controls. The data sovereignty posture is strong, with all infrastructure anchored to AWS eu-west-2 and PostHog EU Cloud, satisfying UK/EU GDPR data residency requirements.

~~Three blocking issues prevent unconditional approval.~~ **[v1.1 update: BLOCKING-01 resolved — see below]** Two blocking issues remain. The original RTO discrepancy (BLOCKING-01) has been resolved: the HLD was conflating two distinct DR scenarios. The standard Multi-AZ failover RTO has now been modelled at < 15 minutes (RDS failover 60–120s + ECS task replacement 60s + ALB registration 30s + connection pool re-establishment 10s ≈ 4–6 minutes typical, < 15 minutes worst-case), which satisfies NFR-A-002 (≤ 30 minutes). The previously stated < 4-hour figure applies only to the separate point-in-time restore (PITR) data corruption recovery scenario. ARC-001-DIAG-001-v1.0 has been updated to reflect this separation.

The remaining two blocking issues are: the ElastiCache layer, specified as a single node for Year 1 with cluster mode disabled, which makes the cache a single point of failure for JWT session invalidation, API rate limiting, and SSE pub/sub fan-out; and the incomplete observability strategy at HLD stage — Grafana Cloud is selected as a vendor but SLI/SLO definitions, distributed tracing approach, structured log schema, and alerting runbooks are absent.

Five advisory items should be addressed during detailed design, including resilience pattern documentation (circuit breakers, bulkhead isolation), WAF timeline, full-text search architecture for Year 3 scale, TOTP secret storage, and PostHog conditional loading for PECR compliance.

### 2.2 Key Strengths

- **Comprehensive security design**: Defence-in-depth with zero-trust patterns, DB-role-level RBAC, immutable audit log, Secrets Manager rotation, and TLS 1.3 throughout — exceeds most HLD submissions at this stage
- **Sound technology choices**: Fastify v5 (2.3× Express throughput), Next.js 15 RSC, Valkey pub/sub for SSE fan-out, and OpenTofu IaC — all evidence-based decisions documented in ADR-001 and DIAG-001
- **Strong data governance**: GDPR data residency, PII handling policy, DPIA requirement flagged, PostHog anonymisation via UUID distinct_id and CSS `ph-no-capture`, ActivityLog write-once with month partitioning plan — demonstrates mature data governance thinking
- **Stateless, horizontally scalable application tier**: ECS Fargate with auto-scaling from min 2 to max 20 tasks, stateless API layer using Valkey for shared session state — scales correctly to Year 3 projections
- **Clear ADR discipline**: SSE vs WebSocket decision documented with performance rationale, technology choices with Wardley evolution scoring — good architectural transparency

### 2.3 Key Concerns

- ~~**RTO discrepancy (BLOCKING)**~~ **✅ RESOLVED (v1.1)**: Standard failover RTO modelled at < 15 min (meets NFR-A-002 ≤ 30 min); PITR restore (< 4 hr) is a separate data corruption DR scenario — DIAG-001 updated
- **ElastiCache SPOF in Year 1 (BLOCKING)**: Single-node cache with no replication is a single point of failure for security-critical functions (JWT blocklist, rate limiting, SSE delivery)
- **Observability incomplete at HLD stage (BLOCKING)**: No SLI/SLO definitions, no distributed tracing strategy, no alerting thresholds — Principle 5 validation gates cannot pass without these
- **Resilience patterns not documented (ADVISORY)**: Circuit breakers, retry policies, and bulkhead isolation required by NFR-A-003 are not visible in the HLD
- **WAF deferred to Phase 2 (ADVISORY)**: OWASP Top 10 NFR-SEC-006 cannot be fully satisfied at launch without Layer 7 WAF protection

### 2.4 Conditions for Approval

**MUST Address Before Detailed Design**:

1. **[BLOCKING-01]** ✅ **RESOLVED (v1.1 — 2026-03-11)**: RTO analysis completed and DIAG-001 updated. Standard Multi-AZ failover RTO is < 15 minutes (meets NFR-A-002 ≤ 30 min). The < 4-hour figure in the original HLD was misclassified — it correctly applies only to the PITR data corruption restore scenario, which is now documented separately. No architecture change required.
2. **[BLOCKING-02]**: Enable ElastiCache Multi-AZ replication group from Day 1 (not Year 3). The cache must not be a SPOF for JWT blocklist (security), rate limiting (abuse prevention), or SSE fan-out (availability).
3. **[BLOCKING-03]**: Add observability architecture section to HLD — minimum: SLI/SLO definitions, distributed tracing approach (OpenTelemetry recommended), structured log schema, and Grafana Cloud dashboard/alerting strategy with actionable runbooks.

**SHOULD Address During Detailed Design**:

1. **[ADVISORY-01]**: Document resilience patterns — circuit breaker configuration for Stripe, Postmark/SES, and PostHog; retry policy with exponential backoff and jitter; bulkhead isolation for notification processing; timeout budgets for all outbound calls.
2. **[ADVISORY-02]**: Define WAF timeline — either deploy AWS WAF at launch (OWASP managed rule group) or document compensating controls that satisfy NFR-SEC-006 until Phase 2.
3. **[ADVISORY-03]**: Document full-text search strategy for FR-010 at Year 3 scale (2M tasks) — either PostgreSQL `pg_trgm`/`tsvector` with GIN index analysis at 2M rows, or add OpenSearch/Elasticsearch to the architecture roadmap.
4. **[ADVISORY-04]**: Document TOTP MFA secret storage — TOTP seeds are sensitive credentials; specify encryption approach (KMS envelope encryption) and storage location in HLD.
5. **[ADVISORY-05]**: Document PostHog conditional loading strategy for NFR-C-003 (UK PECR) — PostHog SDK must be blocked until cookie consent opt-in is confirmed; specify implementation approach (e.g. `posthog.opt_in_capturing()` pattern).

### 2.5 Recommendation

- [ ] **APPROVED**: Proceed to detailed design with no conditions
- [x] **APPROVED WITH CONDITIONS**: Proceed after addressing BLOCKING-02 and BLOCKING-03 (BLOCKING-01 resolved in v1.1)
- [ ] **APPROVED WITH ADVISORIES**: Proceed but address advisory items in DLD
- [ ] **REJECTED**: Significant rework required; resubmit revised HLD for review

**Target Resolution Date for Blocking Items**: 2026-03-25

---

## 3. Architecture Principles Compliance

### 3.1 Principle Compliance Checklist

| Principle ID | Principle Name | Category | Status | Comments |
|--------------|----------------|----------|--------|----------|
| **P-1** | Scalability and Elasticity | Strategic | ✅ Compliant | ECS Fargate auto-scaling; stateless API; Valkey for shared state |
| **P-2** | Resilience and Fault Tolerance | Strategic | ⚠️ Partial | Multi-AZ components good; circuit breakers/bulkheads not documented |
| **P-3** | Interoperability and Integration | Strategic | ✅ Compliant | REST API with OpenAPI 3.0 via @fastify/swagger; versioned JWT; no cross-boundary DB access |
| **P-4** | Security by Design (NON-NEGOTIABLE) | Strategic | ✅ Compliant | Defence-in-depth; JWT RS256; RBAC; TLS 1.3; KMS; Secrets Manager |
| **P-5** | Observability and Operational Excellence | Strategic | ❌ Non-Compliant | Grafana Cloud selected but SLI/SLO, tracing, log schema absent — BLOCKING |
| **P-6** | Data Sovereignty and Governance | Data | ✅ Compliant | eu-west-2; PostHog EU Cloud; PII classification; DPIA flagged |
| **P-7** | Data Quality and Lineage | Data | ⚠️ Partial | ActivityLog immutability good; data quality rules and lineage metadata not specified |
| **P-8** | Single Source of Truth | Data | ✅ Compliant | PostgreSQL as SOR; Valkey explicitly ephemeral/cache |
| **P-9** | Loose Coupling | Integration | ✅ Compliant | All inter-system communication via HTTP APIs; no shared databases |
| **P-10** | Asynchronous Communication | Integration | ⚠️ Partial | SSE pub/sub for real-time; async notification dispatch not explicitly designed |
| **P-11** | Performance and Efficiency | Quality | ✅ Compliant | Fastify throughput; Valkey caching; CDN for assets; DB composite indexes |
| **P-12** | Availability and Reliability | Quality | ⚠️ Partial | Multi-AZ good; RTO modelled at < 15 min ✅ (BLOCKING-01 resolved); ElastiCache single-node SPOF remains — BLOCKING-02 |
| **P-13** | Maintainability and Evolvability | Quality | ✅ Compliant | ADRs documented; TypeScript throughout; clear service boundaries |
| **P-14** | Infrastructure as Code | DevOps | ✅ Compliant | OpenTofu (BSL-safe Terraform fork); all infra as code |
| **P-15** | Automated Testing | DevOps | ⚠️ Partial | GitHub Actions CI/CD; dependency scanning referenced; test pyramid strategy not in HLD |
| **P-16** | Continuous Integration and Deployment | DevOps | ✅ Compliant | GitHub Actions pipeline; automated deployment; rollback via ECS health checks |

**Compliant**: 10 | **Partial**: 5 | **Non-Compliant**: 1

---

### 3.2 Principle Compliance Details

#### P-1: Scalability and Elasticity

**Assessment**: ✅ Compliant

**Evidence**:
- ECS Fargate auto-scaling: API Server min 2 / max 20 tasks (CPU 70% trigger); Web Application min 2 / max 10 tasks
- Stateless application containers — all shared state (sessions, rate limits, pub/sub) in Valkey
- Year 3 capacity model documented: 5,000 concurrent users, 500 req/s, 2M tasks
- Database scaling roadmap: read replica Year 2, ElastiCache cluster mode Year 3, ActivityLog S3 Glacier archive after 3 years

**Concerns**: ElastiCache Year 1 single node limits cache scaling options; addressed under BLOCKING-02.

**Recommendation**: Enable Multi-AZ replication group now; cluster mode upgrade path is already planned.

---

#### P-2: Resilience and Fault Tolerance

**Assessment**: ⚠️ Partial

**Evidence**:
- RDS Multi-AZ with automatic failover (< 60s)
- ElastiCache Multi-AZ planned (currently single node for Year 1 — see BLOCKING-02)
- ECS health checks with task replacement
- SSE EventSource auto-reconnect with `Last-Event-ID` and Redis Streams replay for missed events
- ALB cross-AZ routing

**Concerns**:
- Circuit breakers not visible for external dependencies (Stripe, Postmark/SES, PostHog)
- Retry policy with exponential backoff not documented
- Bulkhead isolation for notification processing not specified
- Timeout budgets for outbound calls not set in HLD
- RTO gap: NFR-A-002 requires 30 minutes; HLD documents < 4 hours (see BLOCKING-01)

**Recommendation**: Add resilience patterns section to HLD (ADVISORY-01). Resolve RTO discrepancy (BLOCKING-01).

---

#### P-3: Interoperability and Integration

**Assessment**: ✅ Compliant

**Evidence**:
- REST API with OpenAPI 3.0 auto-generated by `@fastify/swagger`
- JWT RS256 with JWKS endpoint at `/.well-known/jwks.json` — standard, consumer-friendly
- All external integrations use HTTPS + published protocols: OIDC, Stripe REST + HMAC webhooks, Postmark REST, PostHog REST
- No direct database access across system boundaries
- Key API endpoints documented with method, auth, and purpose

**Concerns**: API versioning strategy not explicitly stated in HLD — DLD should specify URL versioning (e.g. `/api/v1/`) or header-based versioning approach.

---

#### P-4: Security by Design

**Assessment**: ✅ Compliant

**Evidence**:
- JWT RS256 asymmetric signing — public key at JWKS endpoint; no symmetric key exposure risk
- RBAC enforced at two layers: API middleware (`WorkspaceMember.role`) and DB role (`api_user`, `audit_user`)
- AWS Secrets Manager with 30-day rotation; no plaintext secrets in env vars or code
- TLS 1.3 enforced on all connections; HSTS headers; ALB HTTPS-only
- AES-256 via AWS KMS for RDS and ElastiCache at-rest encryption
- ActivityLog immutability via DB-level `REVOKE UPDATE/DELETE` on `activity_log` table
- Rate limiting via Valkey sliding window (per-workspace and per-user)
- HMAC signature validation for Stripe webhooks
- CORS restricted to `https://app.quento.app` only

**Concerns**:
- TOTP MFA secret storage not documented — TOTP seeds must be encrypted at rest; specify KMS envelope encryption approach (ADVISORY-04)
- WAF deferred to Phase 2 — OWASP Layer 7 protection absent at launch (ADVISORY-02)
- Threat model document not produced — Principle 4 validation gate requires one; should be created before DLD

**Recommendation**: Threat model should be produced as a separate artefact before DLD. WAF timeline should be resolved (ADVISORY-02).

---

#### P-5: Observability and Operational Excellence

**Assessment**: ❌ Non-Compliant

**Evidence present**:
- Grafana Cloud listed as observability vendor in vendor profile
- ActivityLog provides task-level audit trail
- GitHub Actions CI/CD pipeline documented

**Missing (Principle 5 validation gates not satisfied)**:
- No SLI (Service Level Indicators) defined — e.g. request latency p95, error rate, SSE delivery latency
- No SLO (Service Level Objectives) defined — e.g. p95 < 200ms for 99.5% of requests in a rolling 7-day window
- No structured log schema specified — correlation IDs, request tracing, and security event log format not described
- No distributed tracing strategy — OpenTelemetry (or equivalent) not mentioned; critical for debugging SSE and Valkey pub/sub latency
- No alerting strategy — Grafana alerting rules, PagerDuty/OpsGenie integration, and SLO burn-rate alerts absent
- No runbooks — required before production per Principle 5

**Recommendation**: This is BLOCKING-03. Add a dedicated observability architecture section to HLD before DLD proceeds.

---

#### P-6: Data Sovereignty and Governance

**Assessment**: ✅ Compliant

**Evidence**:
- All containers deployed in `AWS eu-west-2` (London)
- PostHog EU Cloud — no EU/UK personal data transits to US PostHog servers
- PII fields identified: `User.email`, `User.display_name`
- UK GDPR erasure: email → null, display_name → "Deleted User"; ActivityLog actor_id retained but PII zeroed
- DPIA flagged as required before production launch per GDPR Art. 35
- DPO appointment required before production per GDPR Art. 37
- Data residency: Standard Contractual Clauses noted for third-party processors
- Postmark DPA: 45-day data retention acknowledged

**Concerns**: NFR-C-003 (UK PECR cookie consent for PostHog analytics) not addressed in HLD — see ADVISORY-05.

---

#### P-7: Data Quality and Lineage

**Assessment**: ⚠️ Partial

**Evidence**:
- ActivityLog entity provides task-level event lineage
- Write-once, partitioned ActivityLog with millisecond-precision timestamps
- Data contracts for domain entities defined in ARC-001-DATA-v1.0

**Concerns**:
- Data quality rules not defined (completeness, consistency checks)
- Schema evolution strategy not documented — how will migrations be managed with zero downtime?
- Data lineage metadata for analytics pipeline not specified

**Recommendation**: Address schema migration strategy (zero-downtime PostgreSQL migrations) in DLD.

---

#### P-8: Single Source of Truth

**Assessment**: ✅ Compliant

**Evidence**:
- PostgreSQL (RDS) is the single system of record for all 8 domain entities
- Valkey explicitly documented as ephemeral cache — "ElastiCache data is acceptable data loss"
- No bidirectional synchronisation between stores

---

#### P-9: Loose Coupling

**Assessment**: ✅ Compliant

**Evidence**:
- Web Application communicates with API Server via HTTPS REST + SSE; no direct DB access
- All external integrations via published HTTPS APIs
- Each container owns its own data layer; no cross-service DB sharing

---

#### P-10: Asynchronous Communication

**Assessment**: ⚠️ Partial

**Evidence**:
- SSE + Valkey pub/sub for real-time event fan-out — correctly asynchronous
- PostHog analytics described as "fire-and-forget (async)"

**Concerns**:
- Notification dispatch pipeline (email + in-app) is not explicitly designed as async in HLD
- No queue/worker pattern shown for email delivery, due-date reminder jobs, or data export generation (FR-019)
- Without an async queue, a Postmark/SES failure could block task mutation operations

**Recommendation**: Add async notification queue (e.g. AWS SQS or Valkey list-based queue) to HLD for email dispatch and scheduled reminder jobs. Required for NFR-A-003 bulkhead isolation (ADVISORY-01).

---

#### P-11: Performance and Efficiency

**Assessment**: ✅ Compliant

**Evidence**:
- Fastify v5: 76,000 req/s throughput (2.3× Express); meets NFR-P-004 (200 req/s peak)
- Valkey response caching: 5-minute TTL for task lists; offloads DB read load
- Kysely compiled query cache; composite indexes on `(workspace_id, status)`
- Next.js RSC: zero client JS for static pages; `@next/standalone` optimised bundle
- CloudFront CDN for static assets (satisfies NFR-P-003 < 2s page load)
- NFR-P-001 (p95 < 500ms) supported by Fastify throughput + Valkey caching + DB indexing

---

#### P-12: Availability and Reliability

**Assessment**: ⚠️ Partial

**Evidence**:
- RDS Multi-AZ with automatic failover (< 60s) — exceeds most SLA requirements
- ElastiCache Multi-AZ planned (Year 3; Year 1 single node — BLOCKING-02)
- ECS health checks with automatic task replacement
- ALB cross-AZ load distribution
- NFR-A-001 (99.5% uptime): RDS Multi-AZ + ECS supports this

**Concerns**:
- ~~**BLOCKING-01**~~ **✅ RESOLVED (v1.1)**: Application-level RTO modelled: RDS Multi-AZ failover (60–120s) + ECS health check detection (60s) + new task startup (60s) + ALB registration (30s) + connection pool re-establishment (10s) = ≈ 4–6 minutes typical, < 15 minutes worst-case. Satisfies NFR-A-002 (≤ 30 minutes). The < 4-hour figure in the original HLD applied to the PITR data corruption restore scenario only; DIAG-001 now documents both scenarios separately.
- **BLOCKING-02**: ElastiCache single node Year 1 — if cache fails, JWT blocklist cannot be checked (security bypass risk), rate limiting ceases (abuse risk), SSE pub/sub fails (availability impact). This must be Multi-AZ from Day 1.

---

#### P-13: Maintainability and Evolvability

**Assessment**: ✅ Compliant

**Evidence**:
- ADR-001 documents SSE real-time decision with alternatives considered
- TypeScript throughout (Next.js, Fastify, Kysely) — type safety enables confident refactoring
- Kysely preferred over Prisma with documented rationale
- Clear service boundary: Web Application (presentation) / API Server (domain) / Task Database (persistence)
- Architecture Decision Records discipline established

---

#### P-14: Infrastructure as Code

**Assessment**: ✅ Compliant

**Evidence**:
- OpenTofu (MPL 2.0, BSL-safe Terraform fork) for all AWS infrastructure
- Infrastructure versioned alongside application code
- GitHub Actions automates infrastructure deployment

---

#### P-15: Automated Testing

**Assessment**: ⚠️ Partial

**Evidence**:
- GitHub Actions CI/CD with dependency scanning referenced
- `jest-axe` for automated WCAG testing in CI (NFR-U-002)
- Lighthouse CI in deployment pipeline (NFR-P-003)
- k6 load testing referenced (NFR-P-004)

**Concerns**:
- Test pyramid strategy (unit/integration/e2e split) not defined in HLD
- SAST tooling not specified (NFR-SEC-005 requires SAST on every PR)
- Container image vulnerability scanning tooling not specified (Trivy or equivalent)

**Recommendation**: Specify SAST tooling (e.g. CodeQL, Snyk Code) and container scanning (e.g. Trivy) in DLD.

---

#### P-16: Continuous Integration and Deployment

**Assessment**: ✅ Compliant

**Evidence**:
- GitHub Actions pipeline documented in vendor profile
- ECS rolling deployment with health check-based rollback
- OpenTofu manages infrastructure changes through pipeline
- Multi-environment deployment (dev, staging, prod inferred from IaC approach)

---

## 4. Requirements Coverage Analysis

### 4.1 Functional Requirements Coverage

| Requirement ID | Requirement Summary | Addressed | Design Element | Assessment |
|----------------|---------------------|-----------|----------------|------------|
| FR-001 | User registration (email + OAuth) | ✅ Yes | API Server (JWT RS256), OAuth Provider (OIDC) | ✅ Adequate |
| FR-002 | Authentication and session management | ✅ Yes | API Server (JWT 15min / Refresh 7d), Valkey JWT blocklist | ✅ Adequate |
| FR-003 | User profile management | ⚠️ Partial | API Server + Task Database (User entity) | ⚠️ Avatar storage not shown in HLD — file/object storage (S3) not in container diagram |
| FR-004 | Workspace management | ✅ Yes | API Server, Task Database (Workspace entity) | ✅ Adequate |
| FR-005 | Role-based access control | ✅ Yes | API Server RBAC middleware, DB `api_user` role | ✅ Adequate |
| FR-006 | Task creation | ✅ Yes | API Server, Task Database (Task entity) | ✅ Adequate |
| FR-007 | Task editing | ✅ Yes | API Server, ActivityLog entity | ✅ Adequate |
| FR-008 | Task status workflow | ✅ Yes | API Server, SSE for real-time broadcast | ✅ Adequate |
| FR-009 | Task deletion and archiving | ✅ Yes | API Server, Task Database | ✅ Adequate |
| FR-010 | Task search and filtering | ⚠️ Partial | API Server + Task Database | ⚠️ Full-text search strategy at Year 3 scale (2M tasks) not addressed — see ADVISORY-03 |
| FR-011 | Task comments and mentions | ✅ Yes | API Server, Task Database (Comment entity), SSE notifications | ✅ Adequate |
| FR-012 | Task activity log | ✅ Yes | Task Database (ActivityLog — write-once, DB REVOKE) | ✅ Adequate — strong immutability design |
| FR-013 | Project management | ✅ Yes | API Server, Task Database (Project entity) | ✅ Adequate |
| FR-014 | Project views (List and Kanban) | ✅ Yes | Web Application (Next.js), SSE for Kanban real-time | ✅ Adequate |
| FR-015 | Project member management | ✅ Yes | API Server, Task Database (WorkspaceMember entity) | ✅ Adequate |
| FR-016 | Personal dashboard | ✅ Yes | Web Application, API Server, SSE | ✅ Adequate |
| FR-017 | In-app notifications | ✅ Yes | API Server, SSE (< 5s delivery per requirement), Cache pub/sub | ✅ Adequate |
| FR-018 | Email notifications | ⚠️ Partial | API Server + Email Delivery (Postmark/SES) | ⚠️ Async dispatch architecture not shown — see ADVISORY-01 |
| FR-019 | Data export (CSV/JSON) | ⚠️ Partial | API Server | ⚠️ 60-second export of 10K tasks needs async background job; no worker/queue visible in HLD |
| FR-020 | Workspace admin panel (user mgmt) | ✅ Yes | Web Application, API Server (`/api/workspaces/{id}/members`) | ✅ Adequate |
| FR-021 | Subscription and billing management | ✅ Yes | API Server (Stripe SDK + webhook handler), Stripe external system | ✅ Adequate |
| FR-022 | Team/project progress dashboard | ⚠️ Partial | Web Application, API Server | ⚠️ Aggregate queries on Task Database may be slow without read replica at Year 1 scale |
| FR-023 | Sub-task creation | ✅ Yes | API Server (parent_task_id validation), Task Database (Task.parent_task_id FK) | ✅ Adequate |
| FR-024 | Sub-task status management | ✅ Yes | API Server, ActivityLog (sub-task + parent summary entries), SSE | ✅ Adequate |
| FR-025 | Sub-task assignment and notifications | ✅ Yes | API Server, Email Delivery, SSE | ✅ Adequate |
| FR-026 | Sub-task progress display | ✅ Yes | API Server, SSE real-time update, Web Application | ✅ Adequate |

**Coverage Summary**:
- Fully Covered: 20 / 26 (77%)
- Partially Covered: 6 / 26 (23%) — FR-003, FR-010, FR-018, FR-019, FR-022
- Not Covered: 0 / 26 (0%)

**Gaps Requiring DLD Attention**:
- **FR-003 (avatar storage)**: S3 bucket for user avatars/file attachments not shown in container diagram. Must be added to architecture.
- **FR-010 (search at scale)**: Full-text search strategy for 2M tasks at Year 3 not specified — see ADVISORY-03.
- **FR-018/FR-019 (async jobs)**: Email dispatch and data export require an async background worker pattern; not visible in current HLD. Risk of cascading failure if synchronous.

---

### 4.2 Non-Functional Requirements Coverage

#### Performance Requirements

| NFR ID | Requirement | Target | HLD Approach | Assessment | Comments |
|--------|-------------|--------|--------------|------------|----------|
| NFR-P-001 | Core action response time p95 | < 500ms | Fastify 76K req/s + Valkey 5-min TTL caching + composite DB indexes | ✅ Adequate | Capacity model supports 500 concurrent users at p95 < 500ms |
| NFR-P-002 | p99 response time | < 1,500ms | Same + ECS auto-scaling headroom | ✅ Adequate | — |
| NFR-P-003 | Initial page load (LCP) | < 2s | Next.js RSC zero-JS + CloudFront CDN + `@next/standalone` | ✅ Adequate | Lighthouse CI validates in pipeline |
| NFR-P-004 | API throughput | 200 req/s, < 0.1% error | ECS auto-scaling; Fastify 76K req/s per task | ✅ Adequate | Min 2 ECS tasks × 76K req/s = well above 200 req/s target |

#### Availability and Resilience Requirements

| NFR ID | Requirement | Target | HLD Approach | Assessment | Comments |
|--------|-------------|--------|--------------|------------|----------|
| NFR-A-001 | Monthly uptime SLA | 99.5% | RDS Multi-AZ + ElastiCache (when Multi-AZ) + ECS multi-task | ✅ Adequate | With BLOCKING-02 resolved |
| NFR-A-002 | RPO / RTO | RPO 1hr / RTO 30min | RPO: RDS WAL shipping (near-real-time, sub-5-min lag). RTO: standard failover < 15 min (modelled); PITR restore < 4 hr (separate data corruption scenario) | ✅ Met | **BLOCKING-01 RESOLVED (v1.1)**: Standard failover RTO < 15 min satisfies ≤ 30 min requirement. DIAG-001 updated. |
| NFR-A-003 | Fault tolerance, graceful degradation | Core functions survive external failure | SSE reconnect, Valkey isolation | ⚠️ Partial | Circuit breakers + bulkheads not documented — ADVISORY-01 |
| NFR-A-004 | Multi-AZ deployment | All components in ≥ 2 AZs | RDS Multi-AZ, ECS Fargate multi-AZ, ALB cross-AZ | ⚠️ Partial | ElastiCache Year 1 single node violates this — BLOCKING-02 |

#### Security Requirements

| NFR ID | Requirement | HLD Approach | Assessment | Comments |
|--------|-------------|--------------|------------|----------|
| NFR-SEC-001 | Authentication (email/password + OAuth) | JWT RS256 + OIDC (Google/Microsoft) + TOTP MFA for Admins | ✅ Adequate | TOTP secret storage not documented — ADVISORY-04 |
| NFR-SEC-002 | Authorisation (RBAC) | API middleware + DB-role RBAC (admin/member/viewer + project-level) | ✅ Adequate | Server-side enforced per requirement |
| NFR-SEC-003 | Data encryption (TLS 1.3, AES-256) | TLS 1.3 ALB + RDS AES-256 KMS + ElastiCache TLS | ✅ Adequate | — |
| NFR-SEC-004 | Secrets management | AWS Secrets Manager (30-day rotation); SSM Parameter Store (static config) | ✅ Adequate | — |
| NFR-SEC-005 | Vulnerability management | Dependency scanning in CI/CD; pen test annually | ⚠️ Partial | SAST tool not specified; container scanning tool not specified |
| NFR-SEC-006 | OWASP Top 10 | CORS allowlist; parameterised queries (Kysely); CSP; RBAC; rate limiting | ⚠️ Partial | WAF deferred to Phase 2 — Layer 7 OWASP rule set absent at launch — ADVISORY-02 |

#### Scalability Requirements

| NFR ID | Requirement | HLD Approach | Assessment | Comments |
|--------|-------------|--------------|------------|----------|
| NFR-S-001 | Horizontal scaling (10× without rework) | ECS Fargate auto-scaling (min 2/max 20 API, max 10 web) | ✅ Adequate | Stateless design enables true horizontal scale |
| NFR-S-002 | DB scaling to 2M tasks (50GB) by Year 3 | ActivityLog partitioning; read replica Year 2; S3 Glacier Year 3+ | ✅ Adequate | Partitioning roadmap is detailed and credible |

#### Compliance Requirements

| NFR ID | Requirement | HLD Approach | Assessment | Comments |
|--------|-------------|--------------|------------|----------|
| NFR-C-001 | GDPR compliance | eu-west-2; PII erasure; DPIA flagged; SCCs for processors | ✅ Adequate | FR-019 data export satisfies portability right |
| NFR-C-002 | Audit logging | ActivityLog (write-once, DB REVOKE, 1-year hot / 2-year cold) | ✅ Adequate | Immutability controls at DB level are strong |
| NFR-C-003 | Cookie and tracking consent (PECR) | PostHog EU Cloud | ⚠️ Partial | Conditional PostHog SDK loading not documented — ADVISORY-05 |

---

## 5. Architecture Quality Assessment

### 5.1 System Context Diagram

**Provided**: ✅ Yes (embedded in C4 Container Diagram — external actors and systems identified)

**Assessment**: ✅ Clear

- External actors (Workspace Member, Workspace Admin) clearly identified with roles
- All 4 external systems identified (OAuth Provider, Stripe, Email Delivery, PostHog)
- System boundary clearly defined as `AWS eu-west-2` ECS Fargate cluster

---

### 5.2 Container Diagram (C4 Level 2)

**Provided**: ✅ Yes — ARC-001-DIAG-001-v1.0

**Assessment**: ✅ Clear

**Service Decomposition**:

| Container | Responsibility | Technology | Assessment | Comments |
|-----------|----------------|------------|------------|----------|
| Web Application | SPA delivery, SSR, WCAG 2.1 AA, SSE EventSource | Next.js 15, React, TypeScript | ✅ Adequate | RSC + `@next/standalone` for ECS deployment well-justified |
| API Server | REST API, JWT auth, domain logic, SSE endpoint | Fastify v5, Node.js 22, TypeScript | ✅ Adequate | OpenAPI auto-gen; 2.3× Express throughput; Kysely type-safe SQL |
| Task Database | Persistent storage — 8 entities, self-referential Task FK | PostgreSQL 16, RDS Multi-AZ | ✅ Adequate | Multi-AZ for HA; partitioning roadmap for ActivityLog |
| Cache and Pub/Sub | JWT blocklist, rate limiting, SSE fan-out | Valkey 8, ElastiCache | ⚠️ Partial | Year 1 single node is SPOF — BLOCKING-02 |

**Concerns**:
- File/object storage (S3) not shown — required for FR-003 user avatars and FR-019 export generation
- Background worker for async jobs (email dispatch, due-date reminders, data export) not shown — should be a separate container or ECS task definition in the diagram

---

### 5.3 Technology Stack Review

| Layer | Technology | Assessment | Comments |
|-------|------------|------------|----------|
| Frontend | Next.js 15 App Router, React 19, TypeScript | ✅ Approved | RSC, i18n, jest-axe for WCAG; `@next/standalone` for ECS |
| API Framework | Fastify v5, Node.js 22, TypeScript | ✅ Approved | Performance-justified choice; built-in schema validation |
| ORM/Query | Kysely — type-safe SQL query builder | ✅ Approved | Zero-overhead; full TypeScript inference; preferred over Prisma at scale |
| Primary Database | PostgreSQL 16, AWS RDS Multi-AZ | ✅ Approved | Industry standard; ACID; rich JSON for ActivityLog metadata |
| Cache/Pub-Sub | Valkey 8, AWS ElastiCache | ✅ Approved | BSL-safe Redis fork; native pub/sub; managed service |
| IaC | OpenTofu (MPL 2.0, BSL-safe Terraform fork) | ✅ Approved | BSL licence risk of Terraform mitigated; full AWS provider support |
| CI/CD | GitHub Actions | ✅ Approved | Standard; integrates with GitHub PRs; rich action ecosystem |
| Observability | Grafana Cloud | ⚠️ Vendor selected | Strategy not yet defined — BLOCKING-03 |
| Identity | Google OIDC, Microsoft OIDC | ✅ Approved | Commodity; no custom auth server needed |
| Payments | Stripe | ✅ Approved | Industry standard SaaS billing |
| Email | Postmark (Year 1), AWS SES (Year 2+) | ✅ Approved | Postmark for delivery reputation at launch; SES cost optimisation at scale |
| Analytics | PostHog EU Cloud | ✅ Approved | Open-source; EU GDPR compliance; feature flags for progressive rollout |

**Technology Risks**:
- Fastify plugin ecosystem smaller than Express — team requires onboarding time (documented in ADR-001; acceptable)
- Next.js 15 App Router Server/Client component boundary requires team training — documented; acceptable
- OpenTofu community smaller than HashiCorp Terraform — lower risk as syntax is compatible

---

### 5.4 Data Architecture

**Data Model**: ✅ Comprehensive data model exists in ARC-001-DATA-v1.0

**Key Entities**:

| Entity | Storage | Assessment | Comments |
|--------|---------|------------|----------|
| User (PII) | PostgreSQL — email, display_name | ✅ Adequate | Erasure: email → null, display_name → "Deleted User" |
| Workspace | PostgreSQL | ✅ Adequate | Multi-tenant isolation via workspace_id FK |
| WorkspaceMember | PostgreSQL — role enum | ✅ Adequate | RBAC source of truth |
| Project | PostgreSQL | ✅ Adequate | Container for tasks |
| Task (self-referential) | PostgreSQL — parent_task_id FK (nullable) | ✅ Adequate | Max nesting depth 1 enforced at application layer |
| Comment | PostgreSQL | ✅ Adequate | Markdown content; 10-min edit window |
| Notification | PostgreSQL | ✅ Adequate | Read/unread state; SSE delivery |
| ActivityLog (write-once) | PostgreSQL — REVOKE UPDATE/DELETE | ✅ Adequate | DB-level immutability; monthly partitioning planned |

**Data Flow**: ✅ Clear — PII handling table, integration data flows, and SSE event flow documented in DIAG-001.

**Data Governance**:

| Aspect | Addressed | Assessment |
|--------|-----------|------------|
| Data classification | ✅ Yes | PII (email, display_name) in Confidential tier |
| Data residency (GDPR) | ✅ Yes | eu-west-2; PostHog EU Cloud |
| Data retention policies | ✅ Yes | 90-day cancellation window; annual archival; 3-year hot data |
| PII handling | ✅ Yes | Erasure strategy documented |
| Backup and recovery | ✅ Yes | Daily snapshot + WAL shipping; 30-day retention; cross-region backup |

---

### 5.5 Integration Architecture

| External System | Pattern | Protocol | Authentication | Assessment |
|----------------|---------|----------|----------------|------------|
| OAuth Provider (Google/Microsoft) | OIDC authorisation code flow | HTTPS | OIDC `id_token` | ✅ Adequate |
| Stripe | REST API + signed webhooks | HTTPS | API Key + HMAC `Stripe-Signature` | ✅ Adequate |
| Postmark / AWS SES | REST API | HTTPS | API Key | ⚠️ Partial — async dispatch pattern not shown |
| PostHog Analytics | REST API | HTTPS | API Key | ✅ Adequate — fire-and-forget; EU Cloud for GDPR |

**API Design**:
- [x] OpenAPI 3.0 specifications generated via `@fastify/swagger`
- [x] RESTful design principles followed
- [ ] Versioning strategy — not explicitly defined; should be specified in DLD
- [x] Rate limiting addressed — Valkey sliding window per-workspace and per-user
- [ ] Error response format standardised — not specified in HLD; define in DLD

---

## 6. Security Architecture Review

### 6.1 Threat Model

**Threat Model Provided**: ❌ No — not produced as a separate artefact

**Assessment**: ⚠️ Partial — security controls are comprehensive but the formal threat model required by Principle 4 validation gate has not been produced.

**Key Threats Identified in Design** (inferred from controls):

| Threat | Mitigation in HLD | Assessment |
|--------|-------------------|------------|
| JWT token theft / replay | Short TTL (15min) + Valkey blocklist + refresh token rotation | ✅ Mitigated |
| SQL injection | Kysely parameterised queries; no raw SQL with user input | ✅ Mitigated |
| CSRF on Stripe webhooks | HMAC `Stripe-Signature` validation | ✅ Mitigated |
| Cross-site scripting (XSS) | CORS allowlist; CSP headers | ✅ Mitigated |
| SSRF via external API calls | Not explicitly addressed — should be in threat model | ⚠️ Gap |
| Privilege escalation via role change | Re-authentication required for billing/role changes; Admin action required | ✅ Mitigated |
| SSE stream hijacking | JWT authentication required for `/api/events`; workspace-namespaced channels | ✅ Mitigated |
| Stripe webhook replay attack | HMAC signature + timestamp tolerance (Stripe default: 300s) | ✅ Mitigated |
| Data exfiltration via export | Export scoped to requesting user's accessible tasks | ✅ Mitigated |
| DDoS | WAF planned Phase 2; rate limiting via Valkey now; ALB limits | ⚠️ Partial — WAF absent at launch |

**Recommendation**: Produce formal threat model (STRIDE or LINDDUN for SaaS) before DLD. Specific gap: SSRF threat from outbound HTTP calls (Postmark, PostHog, Stripe) should be analysed.

---

### 6.2 Security Controls

#### Authentication and Authorisation

| Control | Requirement | HLD Approach | Assessment |
|---------|-------------|--------------|------------|
| User authentication | Email/password + OAuth + TOTP MFA for Admins | JWT RS256 + OIDC + TOTP (storage not documented) | ⚠️ TOTP storage gap — ADVISORY-04 |
| Session management | 15-min access token + 7-day refresh | JWT + Valkey blocklist + HttpOnly cookie | ✅ Adequate |
| RBAC authorisation | Admin / Member / Viewer at workspace + project level | API middleware + DB role | ✅ Adequate |
| Service authentication | API to external services | API keys in Secrets Manager; HMAC for webhooks | ✅ Adequate |

#### Network Security

| Control | HLD Approach | Assessment |
|---------|--------------|------------|
| Network segmentation | Public subnet (ALB), private subnet (ECS + RDS + ElastiCache) | ✅ Adequate |
| Ingress protection | ALB HTTPS-only + HSTS; WAF Phase 2 | ⚠️ WAF gap — ADVISORY-02 |
| Egress control | ECS tasks in private subnet; NAT Gateway for outbound | ✅ Adequate (inferred from VPC design) |
| Zero Trust | JWT verification on every API call; no network implicit trust | ✅ Adequate |

#### Data Protection

| Control | HLD Approach | Assessment |
|---------|--------------|------------|
| Encryption in transit | TLS 1.3 (ALB, RDS, ElastiCache, all external APIs) | ✅ Adequate |
| Encryption at rest | AES-256 via AWS KMS (RDS, ElastiCache) | ✅ Adequate |
| Secrets management | AWS Secrets Manager (30-day rotation) + SSM Parameter Store | ✅ Adequate |
| PII masking/pseudonymisation | PostHog UUID distinct_id (no email); CSS `ph-no-capture` | ✅ Adequate |

---

### 6.3 Compliance Mapping

| Requirement | Control | Assessment | Gap |
|-------------|---------|------------|-----|
| GDPR Art. 32 (Security) | Encryption, RBAC, audit logging | ✅ Adequate | — |
| GDPR Art. 17 (Right to erasure) | email → null, display_name → "Deleted User", 90-day retention window | ✅ Adequate | — |
| GDPR Art. 20 (Data portability) | FR-019 JSON/CSV export | ✅ Adequate | — |
| GDPR Art. 35 (DPIA) | DPIA flagged as required before launch | ✅ Flagged | DPIA not yet created — run `/arckit:dpia` |
| UK PECR (cookie consent) | PostHog EU Cloud selected | ⚠️ Partial | Conditional SDK loading not documented — ADVISORY-05 |
| NFR-C-002 (Audit logging) | ActivityLog write-once, 1yr hot / 2yr cold | ✅ Adequate | — |

---

## 7. Scalability and Performance Architecture

### 7.1 Scalability Strategy

| Aspect | HLD Approach | Target | Assessment |
|--------|--------------|--------|------------|
| Horizontal scaling (compute) | ECS Fargate auto-scaling: API max 20, Web max 10 | 5,000 concurrent users (Year 3) | ✅ Adequate |
| Database scaling (reads) | Read replica Year 2; partitioned ActivityLog | 2M tasks, 500K users (Year 3) | ✅ Adequate |
| Cache scaling | Single node Year 1; cluster mode Year 3 | — | ⚠️ BLOCKING-02 — single node SPOF |
| ActivityLog partitioning | Monthly range partitioning by `created_at`; S3 Glacier Year 3 | 100M+ rows at Year 3 | ✅ Adequate |
| Full-text search | Not specified | 2M tasks (Year 3) | ⚠️ ADVISORY-03 |

**Growth Projections Addressed**: ✅ Yes — Year 1 / 2 / 3 capacity model documented.

**Bottlenecks Identified**:
- ElastiCache single node (Year 1): SPOF — BLOCKING-02
- ActivityLog table: Year 3 partitioning planned; adequate
- Dashboard aggregate queries (FR-022): may be slow at Year 1 without read replica — consider adding Year 1 read replica for analytics queries

---

### 7.2 Performance Optimisation

| Optimisation | HLD Approach | Assessment |
|--------------|--------------|------------|
| API response time | Fastify + Valkey caching (5-min TTL) + Kysely compiled queries | ✅ Adequate |
| DB query optimisation | Composite indexes on `(workspace_id, status)`; indexed FKs | ✅ Adequate |
| Static assets | CloudFront CDN via `@next/standalone` | ✅ Adequate |
| Real-time delivery | Valkey pub/sub < 1ms + SSE write < 5ms | ✅ Adequate |
| Search queries | Not specified at HLD stage | ⚠️ See ADVISORY-03 |

**Performance Testing Plan**: ⚠️ Referenced (k6, Lighthouse CI) but not detailed in HLD — should be specified in DLD.

---

## 8. Resilience and Disaster Recovery

### 8.1 Resilience Patterns

| Pattern | Documented | Assessment | Comments |
|---------|------------|------------|----------|
| Circuit breaker | ❌ Not documented | ⚠️ Gap | Required by NFR-A-003 and Principle 2 — ADVISORY-01 |
| Retry with exponential backoff | ❌ Not documented | ⚠️ Gap | Required for Stripe, email, PostHog calls — ADVISORY-01 |
| Timeout on all network calls | ❌ Not documented | ⚠️ Gap | NFR-A-003 specifies max 5-second timeout — ADVISORY-01 |
| Bulkhead isolation | ❌ Not documented | ⚠️ Gap | Notification processing must not block task operations — ADVISORY-01 |
| Graceful degradation | ✅ Inferred | ⚠️ Partial | PostHog fire-and-forget is correct; email degradation not explicit |
| Health checks | ✅ Yes | ✅ Adequate | ECS health checks with task replacement |
| SSE reconnection | ✅ Yes | ✅ Adequate | EventSource + `Last-Event-ID` + Redis Streams replay |

**Single Points of Failure**:
- ElastiCache Year 1 single node — **BLOCKING-02**; must be resolved before DLD

---

### 8.2 High Availability Architecture

| Aspect | HLD Approach | Target SLA | Assessment |
|--------|--------------|------------|------------|
| Multi-AZ deployment (compute) | ECS Fargate across AZs via ALB | 99.5% | ✅ Adequate |
| Database HA | RDS Multi-AZ automatic failover (< 60s) | 99.5% | ✅ Adequate |
| Cache HA | Single node Year 1 / Multi-AZ Year 3 | 99.5% | ❌ Blocked — BLOCKING-02 |
| Stateless services | Valkey for all shared state | N/A | ✅ Adequate |
| ALB health monitoring | ALB health checks + ECS task replacement | N/A | ✅ Adequate |

**Availability SLA Target**: 99.5% (3.6 hours downtime/month)

**Calculated Availability** (with BLOCKING-02 unresolved):
- RDS Multi-AZ: ~99.95%
- ElastiCache single node: ~99.5% — this is the binding constraint, and a cache failure has security implications beyond availability (JWT blocklist)
- ECS Fargate (multi-task): ~99.99%
- Combined: Limited by ElastiCache single node availability — resolving BLOCKING-02 improves this materially

---

### 8.3 Disaster Recovery

| Aspect | Requirement (NFR-A-002) | HLD Approach | Assessment |
|--------|------------------------|--------------|------------|
| RPO | 1 hour | RDS WAL shipping (sub-5-min lag in practice) + daily snapshot | ✅ RPO met — continuous WAL shipping provides sub-1-hour RPO |
| RTO — Standard failover | 30 minutes | RDS Multi-AZ failover (60–120s) + ECS replacement (60s) + ALB registration (30s) + pool re-establish (10s) = < 15 min typical | ✅ **BLOCKING-01 RESOLVED (v1.1)** — < 15 min typical; < 15 min worst-case; satisfies ≤ 30 min requirement |
| RTO — Data corruption (PITR) | N/A (separate scenario) | RDS point-in-time restore — < 4 hours | ✅ Noted — this is the logical failure recovery path, not the standard availability RTO |
| Backup strategy | Daily + continuous WAL | RDS automated backups (daily snapshot + WAL) | ✅ Adequate |
| Backup retention | 30 days | RDS snapshot retention: 30 days | ✅ Adequate |
| Geographic backup | Cross-region | Backups in different region from production | ✅ Adequate |
| DR testing | Quarterly (implied by Principle 12) | Not documented | ⚠️ DR drill procedure not specified |

**DR Runbook Provided**: ❌ Deferred — must be created before production launch.

---

## 9. Operational Architecture

### 9.1 Observability

**Assessment**: ❌ Incomplete — **BLOCKING-03**

| Aspect | HLD Approach | Assessment | Comments |
|--------|--------------|------------|----------|
| Logging | Not specified | ❌ Gap | Structured log schema, correlation IDs, log levels not defined |
| Metrics | Not specified | ❌ Gap | SLI definitions absent (request latency p95, error rate, SSE delivery time) |
| Tracing | Not specified | ❌ Gap | Distributed tracing approach (OpenTelemetry) not mentioned |
| Dashboards | Grafana Cloud selected | ⚠️ Partial | Vendor selected; dashboard structure not defined |
| Alerting | Not specified | ❌ Gap | SLO burn-rate alerts, PagerDuty/OpsGenie integration not defined |

**SLI/SLO Defined**: ❌ Not yet — required before DLD (BLOCKING-03)

**Runbooks for Common Issues**: ❌ Not yet — required before production launch

**Recommendation**: Add the following to HLD before DLD:
1. SLI definitions aligned to NFR targets (p95 latency, error rate, SSE delivery latency, DB query time)
2. SLO targets (e.g. p95 API latency < 500ms for 99% of 7-day rolling windows)
3. OpenTelemetry instrumentation plan (trace context propagation across Fastify → PostgreSQL → Valkey)
4. Structured log schema (correlation ID, workspace_id, user_id, request_id, trace_id)
5. Grafana dashboard structure and key panels
6. Alert definitions and runbook links

---

### 9.2 Deployment Architecture

| Aspect | HLD Approach | Assessment |
|--------|--------------|------------|
| Infrastructure as Code | OpenTofu — all AWS infrastructure | ✅ Adequate |
| CI/CD pipeline | GitHub Actions: build → test → deploy | ✅ Adequate |
| Deployment strategy | ECS rolling deployment with health checks | ✅ Adequate |
| Rollback | ECS health check-based automatic task replacement | ✅ Adequate |
| Environment parity | Dev / Staging / Prod via OpenTofu | ✅ Adequate (inferred) |

**Deployment Downtime**: Near-zero (ECS rolling deployment with health check gates)

---

### 9.3 Supportability

| Aspect | Addressed | Assessment |
|--------|-----------|------------|
| Operational runbooks | ❌ Not yet | Deferred to pre-production — required by Principle 5 |
| Incident response plan | ❌ Not yet | Required before production launch |
| On-call procedures | ❌ Not yet | Required before production launch |
| Capacity planning | ✅ Yes | Year 1/2/3 capacity model documented in DIAG-001 |

---

## 10. Cost Architecture

### 10.1 Cost Estimation

**Note**: Full FinOps analysis is out of scope for this HLD review. BR-003 requires infrastructure costs below 20% of MRR at all scale milestones. A separate FinOps review (`/arckit:finops`) should be conducted.

**Indicative Assessment**:
- ECS Fargate (2 API + 2 Web tasks at 0.5 vCPU / 1GB): ~$40-60/month at launch
- RDS Multi-AZ (db.t3.medium): ~$80-100/month
- ElastiCache (single node Year 1 / replication group): ~$30-50/month
- ALB + data transfer: ~$20-30/month
- CloudFront CDN: negligible at Year 1 volume
- **Estimated Total Year 1**: ~$170-240/month — well within 20% of £15K MRR (£3,000/month threshold)

**Assessment**: ✅ Within budget at Year 1 — managed services and right-sized compute support sub-linear cost scaling.

### 10.2 FinOps Practices

| Practice | Addressed | Assessment |
|----------|-----------|------------|
| Resource tagging | ❌ Not specified | Should be defined before infrastructure is deployed |
| Cost monitoring | ❌ Not specified | AWS Cost Explorer + Grafana cost panel recommended |
| Budget alerts | ❌ Not specified | AWS Budgets alerts should be configured at launch |

---

## 11. Issues and Recommendations

### 11.1 Critical Issues — BLOCKING

Issues that **MUST** be resolved before proceeding to detailed design.

| ID | Issue | Impact | Recommendation | Owner | Target Date |
|----|-------|--------|----------------|-------|-------------|
| ~~BLOCKING-01~~ | ~~RTO discrepancy: NFR-A-002 requires ≤ 30 minutes; HLD documents < 4 hours~~ | ~~HIGH~~ | **✅ RESOLVED (v1.1 — 2026-03-11)**: Standard failover RTO modelled at < 15 min (meets ≤ 30 min). The < 4-hour figure was the PITR data corruption restore path — now documented separately in DIAG-001. No architecture change needed. | ArcKit AI | 2026-03-11 |
| BLOCKING-02 | ElastiCache Year 1 single node — SPOF for JWT blocklist (security), rate limiting (abuse), SSE fan-out (availability) | CRITICAL — security and availability impact if cache fails | Enable ElastiCache replication group (Multi-AZ) from Day 1. The cost delta (single → 2-node replication group) is small (~$30/month). Document this decision as an ADR update. | Engineering Lead | 2026-03-25 |
| BLOCKING-03 | Observability strategy absent — no SLI/SLO definitions, no tracing approach, no alerting strategy in HLD | HIGH — Principle 5 validation gates cannot pass; production operational risk | Add observability architecture section to HLD: SLI/SLO definitions, OpenTelemetry instrumentation plan, structured log schema, Grafana dashboard structure, and alerting runbook links. | Engineering Lead | 2026-03-25 |

---

### 11.2 High Priority Issues — ADVISORY

Issues that **SHOULD** be addressed, preferably in DLD.

| ID | Issue | Impact | Recommendation | Owner | Target Date |
|----|-------|--------|----------------|-------|-------------|
| ADVISORY-01 | Resilience patterns not documented (circuit breakers, retry, bulkheads, timeouts) | MEDIUM — NFR-A-003 requires these; without them external failures may cascade to core task operations | Add resilience patterns section: circuit breaker config for Stripe/Postmark/PostHog; retry with exponential backoff + jitter; max 5s timeout on all outbound calls; async queue (SQS or Valkey list) for email/notification dispatch | Engineering | 2026-04-10 |
| ADVISORY-02 | WAF deferred to Phase 2 — OWASP Layer 7 protection absent at launch | MEDIUM — NFR-SEC-006 OWASP Top 10 not fully satisfied without Layer 7 WAF | Deploy AWS WAF with AWS Managed Rules (OWASP Core rule set) at launch, or document specific compensating controls that satisfy each OWASP Top 10 item without WAF at launch. | Security Architect | 2026-04-10 |
| ADVISORY-03 | Full-text search strategy not specified for FR-010 at Year 3 scale (2M tasks) | MEDIUM — PostgreSQL full-text search at 2M rows may not meet p95 < 500ms without GIN indexes and query planning analysis | Assess PostgreSQL `tsvector` + GIN index approach at 2M rows, or add OpenSearch/Elasticsearch to the architecture roadmap. Specify in DLD with benchmark data. | Engineering | 2026-04-10 |
| ADVISORY-04 | TOTP MFA secret storage not documented — TOTP seeds are sensitive credentials | MEDIUM — if TOTP seeds are stored in plaintext they represent a critical security exposure | Specify KMS envelope encryption for TOTP seeds stored in the User table or Secrets Manager. Document storage format and access pattern. | Security Architect | 2026-04-10 |
| ADVISORY-05 | PostHog conditional loading not documented for UK PECR compliance (NFR-C-003) | MEDIUM — PostHog must not load before user provides analytics cookie consent; violation of UK PECR | Document PostHog SDK conditional initialisation (e.g. `posthog.opt_out_capturing()` by default; `opt_in_capturing()` on consent). Specify cookie consent banner integration in DLD. | Engineering | 2026-04-10 |

---

### 11.3 Low Priority Items — INFORMATIONAL

| ID | Suggestion | Benefit | Owner |
|----|------------|---------|-------|
| INFO-01 | Add S3 bucket container to architecture diagram for user avatars (FR-003) and data exports (FR-019) | Complete container inventory; unambiguous file storage design | Engineering |
| INFO-02 | Add background worker container (ECS task or Lambda) for async jobs (email dispatch, due-date reminders, data export generation) | Explicit async architecture visible in HLD; satisfies P-10 Asynchronous Communication | Engineering |
| INFO-03 | Consider adding ActivityLog partitioning from Year 1 (not Year 3) | Zero-cost at design time; avoids risky live table partitioning migration at Year 3 | Engineering |
| INFO-04 | Consider deploying RDS read replica from Year 1 for dashboard aggregate queries (FR-022 — team progress dashboard) | Prevents dashboard queries from competing with write operations; supports BR-004 NPS target through UI responsiveness | Engineering |
| INFO-05 | Define API versioning strategy in HLD (e.g. URL path `/api/v1/` or `Accept-Version` header) | Enables backward-compatible evolution without breaking integrations | Engineering |
| INFO-06 | Resource tagging strategy should be defined before first OpenTofu `apply` | Enables cost allocation, environment filtering, and FinOps showback from Day 1 | DevOps |

---

## 12. Review Decision

### 12.1 Final Decision

**Status**: APPROVED WITH CONDITIONS

**Effective Date**: 2026-03-11

**Conditions for Proceeding to Detailed Design**:

1. ~~**[BLOCKING-01]** Reconcile RTO~~ — **✅ RESOLVED (v1.1 — 2026-03-11)**: DIAG-001 updated with application-level RTO model. Standard failover RTO < 15 min; NFR-A-002 (≤ 30 min) confirmed met.
2. **[BLOCKING-02]** ElastiCache Multi-AZ replication group from Day 1 — updated HLD required by 2026-03-25
3. **[BLOCKING-03]** Observability architecture section added to HLD (SLI/SLO, tracing, log schema, alerting) — by 2026-03-25

**Next Steps**:

- [ ] Engineering Lead addresses BLOCKING-01, BLOCKING-02, BLOCKING-03 by 2026-03-25
- [ ] Revised HLD sections submitted for re-review
- [ ] Re-review sign-off meeting (30 min) scheduled for week of 2026-03-25
- [ ] DPIA created using `/arckit:dpia` before production launch
- [ ] Threat model produced before DLD begins
- [ ] Proceed to Detailed Design phase once conditions cleared
- [ ] Advisory items addressed during DLD phase

---

### 12.2 Reviewer Sign-Off

| Reviewer | Role | Decision | Signature | Date |
|----------|------|----------|-----------|------|
| [PENDING] | Lead Reviewer / Enterprise Architect | [ ] Approve [x] Conditional [ ] Reject | _________ | 2026-03-11 |
| [PENDING] | Security Architect | [ ] Approve [ ] Conditional [ ] Reject | _________ | [PENDING] |
| [PENDING] | Domain Architect | [ ] Approve [ ] Conditional [ ] Reject | _________ | [PENDING] |
| [PENDING] | Infrastructure Architect | [ ] Approve [ ] Conditional [ ] Reject | _________ | [PENDING] |
| [PENDING] | Data Architect | [ ] Approve [ ] Conditional [ ] Reject | _________ | [PENDING] |
| [PENDING] | SRE Lead | [ ] Approve [ ] Conditional [ ] Reject | _________ | [PENDING] |

**Unanimous Approval Required**: No — majority approval with Lead Reviewer sign-off required.

**Escalation**: Head of Engineering (Jane Smith) → Architecture Review Board

---

## 13. Appendices

### Appendix A: HLD Document References

| Document | Version | Path |
|----------|---------|------|
| C4 Container Diagram (HLD) | 1.0 | `projects/001-task-management-portal/diagrams/ARC-001-DIAG-001-v1.0.md` |
| ADR-001: Real-Time SSE Strategy | 1.0 | `projects/001-task-management-portal/decisions/ARC-001-ADR-001-v1.0.md` |

### Appendix B: Architecture Principles Document

| Document | Version | Path |
|----------|---------|------|
| Quento1 Enterprise Architecture Principles | 1.0 | `projects/000-global/ARC-000-PRIN-v1.0.md` |

### Appendix C: Requirements Document

| Document | Version | Path |
|----------|---------|------|
| Task Management Portal Requirements | 1.1 | `projects/001-task-management-portal/ARC-001-REQ-v1.1.md` |

### Appendix D: Supporting Artefacts

| Document | Version | Path |
|----------|---------|------|
| Data Model | 1.0 | `projects/001-task-management-portal/ARC-001-DATA-v1.0.md` |
| Data Flow Diagram | 1.0 | `projects/001-task-management-portal/diagrams/ARC-001-DFD-001-v1.0.md` |
| Research Findings | 1.0 | `projects/001-task-management-portal/research/ARC-001-RSCH-v1.0.md` |

### Appendix E: Recommended Next Artefacts

| Action | Command | Priority |
|--------|---------|----------|
| Create DPIA before production launch | `/arckit:dpia` | CRITICAL |
| Produce threat model | `/arckit:secure` | HIGH — required before DLD |
| FinOps cost analysis against BR-003 | `/arckit:finops` | MEDIUM |
| Detailed Design Review once DLD created | `/arckit:dld-review` | After DLD is drafted |

---

## External References

| Document | Type | Source | Key Extractions | Path |
|----------|------|--------|-----------------|------|
| ARC-000-PRIN-v1.0 | Enterprise Architecture Principles | Internal | 16 principles, validation gates, exception process | `projects/000-global/ARC-000-PRIN-v1.0.md` |
| ARC-001-REQ-v1.1 | Requirements | Internal | FR-001 to FR-026, NFR-P/A/S/SEC/C/U series, INT-001 to INT-004 | `projects/001-task-management-portal/ARC-001-REQ-v1.1.md` |
| ARC-001-DIAG-001-v1.0 | C4 Container Diagram (HLD) | Internal | Component topology, security zones, NFR coverage, Wardley positioning | `projects/001-task-management-portal/diagrams/ARC-001-DIAG-001-v1.0.md` |
| ARC-001-ADR-001-v1.0 | Architecture Decision Record | Internal | SSE vs WebSocket decision, Redis pub/sub fan-out rationale | `projects/001-task-management-portal/decisions/ARC-001-ADR-001-v1.0.md` |

---

**Generated by**: ArcKit `/arckit:hld-review` command
**Generated on**: 2026-03-11 GMT
**ArcKit Version**: 4.1.1
**Project**: Task Management Portal (Project 001)
**AI Model**: claude-sonnet-4-6
**Generation Context**: ARC-000-PRIN-v1.0 (16 enterprise principles), ARC-001-REQ-v1.1 (FR/NFR/INT requirements v1.1), ARC-001-DIAG-001-v1.0 (C4 Container HLD), ARC-001-ADR-001-v1.0 (real-time decision)
**v1.1 Change**: BLOCKING-01 resolved — ARC-001-DIAG-001-v1.0 updated to separate standard failover RTO (< 15 min) from PITR data corruption restore (< 4 hr). NFR-A-002 (≤ 30 min RTO) confirmed met by current Multi-AZ architecture.
