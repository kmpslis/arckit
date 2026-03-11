# Detailed Design (DLD) Review: Task Management Portal

> **Template Origin**: Official | **ArcKit Version**: 4.1.1 | **Command**: `/arckit:dld-review`

## Document Control

| Field | Value |
|-------|-------|
| **Document ID** | ARC-001-DLDR-v1.0 |
| **Document Type** | Detailed Design Review |
| **Project** | Task Management Portal (Project 001) |
| **Classification** | PUBLIC |
| **Status** | DRAFT |
| **Version** | 1.0 |
| **Created Date** | 2026-03-11 |
| **Last Modified** | 2026-03-11 |
| **Review Cycle** | On-Demand |
| **Next Review Date** | 2026-04-10 |
| **Owner** | Jane Smith, Head of Engineering |
| **Reviewed By** | PENDING |
| **Approved By** | PENDING |
| **Distribution** | Engineering, Product, Architecture Teams |

## Revision History

| Version | Date | Author | Changes | Approved By | Approval Date |
|---------|------|--------|---------|-------------|---------------|
| 1.0 | 2026-03-11 | ArcKit AI | Initial creation from `/arckit:dld-review` command | PENDING | PENDING |

## Document Purpose

This document records the Architecture Review Board's evaluation of the Detailed Design for Quento1's Task Management Portal (Project 001). The DLD under review is derived from the combined design artefact set: C4 Container Diagram (ARC-001-DIAG-001-v1.0), Data Model (ARC-001-DATA-v1.0), Data Flow Diagram (ARC-001-DFD-001-v1.0), and Architecture Decision Record (ARC-001-ADR-001-v1.0). The review assesses implementation readiness against HLD findings, enterprise architecture principles, and technical requirements in ARC-001-REQ-v1.1 before development sprints begin.

---

## 1. Review Overview

### 1.1 Purpose

This document captures the Architecture Review Board's evaluation of the Detailed Design (DLD) for the Task Management Portal. The DLD must provide implementation-ready specifications — APIs, data schemas, security controls, and operational procedures — so that any developer can begin coding without ambiguity.

### 1.2 Review Context

| Field | Value |
|-------|-------|
| **HLD Review Document** | ARC-001-HLDR-v1.1 |
| **HLD Approval Status** | APPROVED WITH CONDITIONS |
| **HLD Approval Date** | 2026-03-11 |
| **HLD Open Conditions** | BLOCKING-02 (ElastiCache SPOF), BLOCKING-03 (Observability absent) |
| **DLD Artefacts Under Review** | ARC-001-DIAG-001-v1.0, ARC-001-DATA-v1.0, ARC-001-DFD-001-v1.0, ARC-001-ADR-001-v1.0 |

> ⚠️ **Important**: The HLD was approved with two open blocking conditions (BLOCKING-02: ElastiCache SPOF; BLOCKING-03: Observability absent). These conditions are inherited by this DLD review and carried forward as **inherited blockers** (DLD-BLOCKING-01 and DLD-BLOCKING-02). They must be resolved at DLD stage before development can begin.

### 1.3 Review Participants

| Name | Role | Review Focus |
|------|------|--------------|
| PENDING | Lead Reviewer | Overall design quality, completeness, implementation readiness |
| PENDING | Domain Architect | Component design, Fastify API, Kysely SQL, SSE patterns |
| PENDING | Security Reviewer | JWT implementation, TOTP storage, Secrets Manager, RBAC |
| PENDING | Data Architect | PostgreSQL schema, migrations, indexes, GDPR |
| PENDING | SRE / Operations | Observability, alerting, ECS configuration, deployment |
| PENDING | QA Lead | Test strategy, coverage targets, performance test plan |

---

## 2. Executive Summary

### 2.1 Overall Assessment

**Status**: APPROVED WITH CONDITIONS

The Detailed Design artefact set for the Task Management Portal demonstrates a high level of implementation maturity for a pre-development SaaS platform. The technology selections are sound and evidence-based (Fastify v5 for throughput, Next.js 15 App Router for SSR performance, PostgreSQL 16 with Kysely for type-safe data access, Valkey 8 for session and pub/sub). The data model (ARC-001-DATA-v1.0) is thorough — 8 entities, 74 attributes, 12 relationships, with detailed attribute-level validation rules, GDPR classification, retention policies, and index specifications. Security controls are defined at a level of detail appropriate for DLD: JWT RS256 with JWKS endpoint, RBAC at middleware and DB role levels, AWS Secrets Manager with 30-day rotation, AES-256 KMS encryption at rest and TLS 1.3 in transit, ActivityLog write-once enforced at DB level.

Five conditions prevent unconditional approval. Two are inherited from the HLD review (BLOCKING-02: ElastiCache single-node SPOF; BLOCKING-03: Observability strategy absent). Three are new DLD-specific findings: an OpenAPI specification artefact has not been produced (auto-generation via `@fastify/swagger` is confirmed but the spec itself is not available for review); a test strategy is absent (no coverage targets, test pyramid, or performance test plan documented); and the database migration strategy for zero-downtime PostgreSQL deployments is not specified. Five advisory items should be addressed during sprint planning.

Subject to resolution of all five conditions, the design is implementation-ready.

### 2.2 Implementation Readiness Score

| Dimension | Score | Comment |
|-----------|-------|---------|
| Component Design | 82/100 | Well-structured containers; module internals not yet decomposed to component (C4 Level 3) |
| API Design | 74/100 | REST conventions and auth clear; full OpenAPI spec not yet produced as artefact |
| Data Model | 90/100 | Excellent entity definitions with validation rules, indexes, and GDPR classification |
| Security Implementation | 85/100 | Strong controls; TOTP storage and WAF timeline need resolution |
| Integration Design | 80/100 | All 4 INT-xxx integrations addressed; circuit breaker configuration absent |
| Performance Design | 78/100 | NFR-P targets mapped to components; load test plan not documented |
| Operational Readiness | 45/100 | BLOCKING — Observability design absent (BLOCKING-03 inherited from HLD) |
| Testing Strategy | 20/100 | BLOCKING — No test strategy, coverage targets, or performance test plan |
| **Overall** | **69/100** | Conditional approval — resolve 5 blocking conditions to reach implementation readiness |

### 2.3 Conditions for Approval

**MUST Address Before Development Begins**:

1. **[DLD-BLOCKING-01]** *(Inherited: HLD BLOCKING-02)*: Enable ElastiCache Multi-AZ replication group from Day 1. Single-node Valkey is a SPOF for JWT blocklist (security-critical), rate limiting (abuse prevention), and SSE fan-out (availability). DLD must specify the replication group configuration: replica count, node type, and failover behaviour.

2. **[DLD-BLOCKING-02]** *(Inherited: HLD BLOCKING-03)*: Add observability architecture section. Minimum required: (a) SLI definitions (request latency p95, error rate, SSE delivery latency, DB query p95); (b) SLO definitions (e.g. p95 < 200ms for 99.5% of requests over rolling 7 days); (c) OpenTelemetry instrumentation strategy for Fastify, Next.js, and Kysely; (d) structured log schema with correlation ID format; (e) Grafana Cloud dashboard structure and alerting rules with SLO burn-rate alerts; (f) runbook index.

3. **[DLD-BLOCKING-03]**: Produce OpenAPI 3.0 specification artefact. The `@fastify/swagger` plugin auto-generates the spec, but the spec must be exported and committed as an artefact (OpenAPI YAML or JSON). This is the API contract for frontend–backend integration and any future third-party consumers. Minimum required: all endpoints from DIAG-001 API table plus full CRUD coverage for Tasks, Workspaces, Members, Comments, Notifications, Auth, and Webhooks.

4. **[DLD-BLOCKING-04]**: Document test strategy. Minimum required: (a) test pyramid with framework choices (Vitest for unit, Supertest + Testcontainers for integration, Playwright for e2e); (b) coverage targets per layer (unit: ≥ 80%; integration: critical paths; e2e: happy paths for all 26 FRs); (c) performance test plan (k6 or Artillery scenario for 500 concurrent users, 500 req/s sustained, SSE fan-out to 5K connections); (d) security test plan (SAST via CodeQL in GitHub Actions, dependency audit via `npm audit`, OWASP ZAP for DAST pre-production).

5. **[DLD-BLOCKING-05]**: Define database migration strategy. PostgreSQL schema migrations in a running SaaS system require a zero-downtime approach. Specify: (a) migration tool (`node-pg-migrate`, Flyway, or Liquibase); (b) migration file naming convention; (c) zero-downtime migration patterns to use (additive-only changes, expand-contract for column renames, deferred NOT NULL constraints, concurrent index creation with `CREATE INDEX CONCURRENTLY`); (d) rollback strategy for each migration type; (e) migration execution as part of CI/CD pipeline (pre-deploy step, gated by health check).

**SHOULD Address During Sprint Planning**:

1. **[DLD-ADVISORY-01]** *(Inherited: HLD ADVISORY-01)*: Define circuit breaker and resilience patterns for all outbound calls — Stripe (payment-critical), Postmark/SES (notification delivery), PostHog (analytics, fire-and-forget). Specify: library (e.g. `cockatiel`), failure thresholds, half-open probe interval, fallback behaviour per integration, timeout budgets, retry policy (exponential backoff with jitter).

2. **[DLD-ADVISORY-02]** *(Inherited: HLD ADVISORY-02)*: Resolve WAF timeline. Either: (a) deploy AWS WAF at launch with OWASP Core Rule Set managed rule group (adds ~£40/month — justifiable given NFR-SEC-006); or (b) document the compensating controls that provide equivalent OWASP Top 10 protection until Phase 2 WAF deployment.

3. **[DLD-ADVISORY-03]**: Specify ECS task resource allocation — vCPU and memory per task for each container (Web Application, API Server). This is required for OpenTofu resource definitions and cost modelling. Suggested starting points: API Server 0.5 vCPU / 1GB RAM, Web Application 0.25 vCPU / 512MB RAM.

4. **[DLD-ADVISORY-04]** *(Inherited: HLD ADVISORY-04)*: Document TOTP MFA secret encryption. `User.mfa_secret` stores a 32-character base32 seed — highly sensitive. Specify: envelope encryption via AWS KMS (data key encrypts the TOTP seed; encrypted data key stored alongside); key rotation policy; how decryption occurs at authentication time without KMS call latency impact (cached decrypted key with TTL).

5. **[DLD-ADVISORY-05]**: Specify background job / notification dispatch mechanism. The DFD and DIAG reference email notification dispatch but do not specify the dispatch pattern. Options: (a) in-process Fastify background worker with `p-queue`; (b) SQS queue + ECS worker process (recommended for resilience and retry); (c) cron-based polling. For NFR-A-001 (99.9% uptime), SQS is recommended — decouples notification failure from API availability.

### 2.4 Recommendation

- [ ] **APPROVED**: Ready for development
- [x] **APPROVED WITH CONDITIONS**: Address 5 blocking conditions before sprint 1 begins
- [ ] **REJECTED**: Significant rework required

**Target Resolution Date**: 2026-04-10

---

## 3. Component Design Review

### 3.1 Web Application (Next.js 15)

**Purpose**: Server-rendered SPA delivering the task management UI; subscribes to SSE endpoint for real-time updates.

| Aspect | Quality | Comments |
|--------|---------|----------|
| Responsibility & Boundaries | ✅ | Presentation-only; no business logic; delegates all data operations to API Server |
| Technology Justification | ✅ | App Router + RSC for SSR performance (NFR-P-002 LCP < 2s); `@next/standalone` for ECS container |
| Authentication Handling | ✅ | JWT stored as HttpOnly cookie; forwarded as Bearer token to API; no token parsing in frontend |
| SSE Client Design | ✅ | Browser `EventSource` with auto-reconnect; `Last-Event-ID` propagated for event replay |
| WCAG 2.1 AA | ✅ | `jest-axe` for automated accessibility testing in CI |
| i18n | ✅ | `next-intl` specified |
| Component Decomposition (C4 L3) | ⚠️ | Module structure (pages, components, hooks, server actions) not yet documented — acceptable for DLD at SaaS MVP scale; recommended before Sprint 3 |
| Error Handling | ⚠️ | Next.js error boundaries referenced; specific error UI patterns not yet defined |
| Configuration | ✅ | `@next/standalone` container; environment variables via ECS task definition from SSM Parameter Store |

### 3.2 API Server (Fastify v5)

**Purpose**: RESTful API with JWT RS256 auth, domain logic for task/workspace management, SSE endpoint with Valkey pub/sub fan-out.

| Aspect | Quality | Comments |
|--------|---------|----------|
| Responsibility & Boundaries | ✅ | Clear API + domain layer; no presentation logic; data layer abstracted via Kysely |
| Technology Justification | ✅ | Fastify v5 76K req/s; auto-OpenAPI via `@fastify/swagger`; JSON schema validation built-in |
| JWT Implementation | ✅ | RS256 asymmetric signing; JWKS at `/.well-known/jwks.json`; 15-min access token; 7-day refresh HttpOnly cookie |
| RBAC Implementation | ✅ | `WorkspaceMember.role` checked per-endpoint via Fastify guard; DB role `api_user` prevents escalation |
| SSE Implementation | ✅ | Valkey pub/sub channels `ws:workspace:{id}`; `Last-Event-ID` for missed-event replay via Redis Streams |
| Rate Limiting | ✅ | Sliding window per-workspace and per-user via Valkey; limits not yet quantified |
| Stripe Webhook Handling | ✅ | `Stripe-Signature` HMAC validation before processing; idempotent webhook processing implied |
| API Versioning | ⚠️ | `/api/v1/` prefix assumed from endpoint table; explicit versioning strategy not documented in DLD |
| Query Layer | ✅ | Kysely type-safe SQL with compiled query cache; `pg` connection pool max 20 per ECS task |
| Error Handling | ⚠️ | Fastify default error serialisation; structured error response format for API consumers not defined |
| OpenAPI Spec | ❌ | Auto-generated by `@fastify/swagger` but not produced as artefact — DLD-BLOCKING-03 |

#### Fastify Route Design Assessment

| Endpoint | Method | Auth | Coverage | Notes |
|----------|--------|------|----------|-------|
| `/api/v1/tasks` | GET, POST, PATCH, DELETE | JWT RS256 | ✅ | Covers FR-006 to FR-022; pagination strategy needed for GET |
| `/api/v1/events` | GET (SSE) | JWT RS256 | ✅ | `?workspace_id=` query param; covers FR-003 real-time |
| `/api/v1/auth/register` | POST | None | ✅ | Email/password registration; covers FR-001 |
| `/api/v1/auth/login` | POST | None | ✅ | Email/password + TOTP; issues JWT pair |
| `/api/v1/auth/refresh` | POST | Refresh token | ✅ | Rotates refresh token |
| `/api/v1/auth/oauth/{provider}` | GET, POST | None | ✅ | OIDC flow; covers INT-001 |
| `/api/v1/webhooks/stripe` | POST | HMAC | ✅ | Covers INT-002; must be idempotent |
| `/api/v1/workspaces/{id}/members` | GET, POST, DELETE | JWT + admin role | ✅ | Covers FR-004 RBAC |
| `/api/v1/workspaces/{id}/projects` | GET, POST | JWT | Implied | Derived from FR-007 and data model |
| `/api/v1/notifications` | GET, PATCH | JWT | Implied | Mark read; covers FR-005 |
| `/.well-known/jwks.json` | GET | None | ✅ | Public key serving; P-3 compliance |
| `/api/health` | GET | None | ⚠️ | Not listed — required for ECS health checks and ALB target group |

**Gap noted**: `/api/health` liveness/readiness endpoint not listed in DLD. Required for ECS + ALB health check configuration.

### 3.3 Task Database (PostgreSQL 16 RDS Multi-AZ)

**Purpose**: Persistent storage for all 8 domain entities; write-once ActivityLog; RDS Multi-AZ HA.

| Aspect | Quality | Comments |
|--------|---------|----------|
| Entity Model | ✅ | 8 entities, 74 attributes, 12 relationships fully documented in ARC-001-DATA-v1.0 |
| GDPR Classification | ✅ | User entity classified CONFIDENTIAL; PII fields `email` and `display_name` identified |
| Soft Deletes | ✅ | `deleted_at` and `archived_at` timestamps throughout; hard deletion after retention period |
| Self-Referential Task FK | ✅ | `parent_task_id UUID FK` on Task; max depth = 1 enforced at application layer |
| ActivityLog Immutability | ✅ | DB role `audit_user`: INSERT-only; `api_user`: REVOKE UPDATE/DELETE on `activity_log` |
| ActivityLog Partitioning | ✅ | Monthly range partitioning by `created_at` specified; Year 3 cold archive to S3 Glacier |
| Encryption at Rest | ✅ | AES-256 via AWS KMS; RDS encryption enabled |
| Connection Pooling | ✅ | Kysely `pg` pool; max 20 connections per ECS API task |
| Index Strategy | ✅ | FKs indexed; composite indexes on `(workspace_id, status)` for Task queries; GIN for JSONB |
| Migration Strategy | ❌ | Zero-downtime PostgreSQL migration approach not defined — DLD-BLOCKING-05 |
| Read Replica (Year 2) | ✅ | Roadmap documented; Kysely read replica connection string switching specified |
| Query Optimisation | ✅ | Indexed FKs; p95 < 50ms target (NFR-P-004); `EXPLAIN ANALYSE` process referenced |

### 3.4 Cache and Pub/Sub (Valkey 8 ElastiCache)

**Purpose**: JWT session invalidation, rate limiting (sliding window), SSE pub/sub fan-out across ECS instances.

| Aspect | Quality | Comments |
|--------|---------|----------|
| JWT Blocklist | ✅ | On logout/token revocation: `JWT:BLOCKLIST:{jti}` key with TTL = remaining token lifetime |
| Rate Limiting | ✅ | Sliding window `ZADD` + `ZREMRANGEBYSCORE` pattern per workspace and per user |
| SSE Pub/Sub Channels | ✅ | `ws:workspace:{id}` channel namespace; Redis Streams for event replay with `Last-Event-ID` |
| Namespace Isolation | ✅ | Workspace-scoped channel naming prevents cross-tenant event leakage |
| High Availability | ❌ | Single primary node Year 1 = SPOF for all three security/availability functions — DLD-BLOCKING-01 |
| TLS + Auth | ✅ | ElastiCache AUTH token + in-transit encryption specified |
| Cache TTL Strategy | ⚠️ | Task list cache TTL = 5 min; other key TTLs not fully documented (rate limit window, blocklist) |
| Cluster Mode | ✅ | Roadmap to cluster mode at Year 3 documented; currently single-shard |

---

## 4. API Design Review

### 4.1 REST API Assessment

**OpenAPI Spec Produced**: ❌ (auto-generated by `@fastify/swagger`; artefact not yet committed — DLD-BLOCKING-03)

| Aspect | Quality | Comments |
|--------|---------|----------|
| RESTful Principles | ✅ | Resource-oriented endpoints; correct HTTP verbs (GET/POST/PATCH/DELETE); plural nouns |
| JSON Schema Validation | ✅ | Fastify built-in schema validation on request body, query params, and response |
| Authentication per Endpoint | ✅ | JWT RS256 Bearer clearly specified per endpoint; HMAC for Stripe webhook; public for JWKS |
| Rate Limiting | ✅ | Sliding window via Valkey specified; per-workspace and per-user scope |
| API Versioning | ⚠️ | `/api/v1/` prefix implied from endpoint table; explicit versioning strategy (URL path vs header) not documented |
| Pagination | ⚠️ | Not specified for list endpoints (`GET /api/v1/tasks`); required for Year 1 at 10K tasks/workspace |
| Error Response Format | ⚠️ | Fastify default serialisation; client-facing error schema (code, message, field) not standardised |
| Idempotency | ⚠️ | Stripe webhook handler must be idempotent (implied but not specified); no idempotency key for task POST |
| CORS | ✅ | Allowlist `https://app.quento.app`; `credentials: true` for SSE |
| Content Type | ✅ | `application/json` for REST; `text/event-stream` for SSE; specified in endpoint table |
| Health Endpoint | ❌ | `/api/health` not defined — required for ECS health checks |

### 4.2 SSE Endpoint Design

**Endpoint**: `GET /api/v1/events?workspace_id={uuid}`

| Aspect | Quality | Comments |
|--------|---------|----------|
| Authentication | ✅ | JWT RS256 Bearer required; workspace membership validated before subscription |
| Multi-Instance Fan-out | ✅ | Valkey pub/sub `ws:workspace:{id}` — all ECS API instances subscribe; fan-out to SSE clients |
| Missed Event Recovery | ✅ | `Last-Event-ID` header; Redis Streams replay from last seen event ID |
| Client Reconnection | ✅ | Browser `EventSource` auto-reconnects; server reconnects to Valkey pub/sub on restart |
| Event Schema | ⚠️ | Event payload structure (`event:`, `data:`, `id:`) not fully documented in DLD artefacts |
| Timeout/Keepalive | ⚠️ | SSE keepalive heartbeat interval not specified; ALB idle timeout (default 60s) will drop long-lived connections without keepalive |
| Workspace Isolation | ✅ | Channel namespace prevents cross-workspace event leakage |

---

## 5. Data Model Review

### 5.1 Schema Assessment

**Full ERD and entity definitions provided in**: ARC-001-DATA-v1.0

| Entity | E-ID | Assessment | Notes |
|--------|------|------------|-------|
| User | E-001 | ✅ | PII fields classified; bcrypt cost ≥ 12; TOTP secret AES-256 encrypted; soft delete with 90-day PII retention |
| Workspace | E-002 | ✅ | Slug unique; Stripe customer ID stored; subscription status state machine implied |
| WorkspaceMember | E-003 | ✅ | Junction table; role constraint (admin/member/viewer); correct FK indexes |
| Project | E-004 | ✅ | Soft archive; creator FK; workspace scoping |
| Task | E-005 | ✅ | Self-referential `parent_task_id`; status and priority enums; dual FKs for assignee and creator |
| Comment | E-006 | ✅ | Soft delete; `edited_at` tracks in-place edits |
| Notification | E-007 | ✅ | JSONB payload for event context; `read_at` null = unread pattern |
| ActivityLog | E-008 | ✅ | Write-once; JSONB old/new value; partition key `created_at`; DB-level REVOKE on UPDATE/DELETE |

### 5.2 Data Type and Constraint Assessment

| Concern | Status | Detail |
|---------|--------|--------|
| UUID v4 primary keys | ✅ | Consistent; auto-generated; no sequential ID leakage |
| Enum enforcement | ⚠️ | `Task.status` and `Task.priority` constraints: PostgreSQL native ENUM type vs CHECK constraint vs application-layer validation — DLD should specify which approach |
| JSONB usage | ✅ | `Notification.payload` and `ActivityLog.old_value/new_value`: appropriate for schema-less event context; GIN index for JSONB querying |
| Text vs VARCHAR | ⚠️ | Mix of `TEXT` (description, content) and `VARCHAR(n)` — document rationale; in PostgreSQL `TEXT` and `VARCHAR` are equivalent storage but VARCHAR(n) enforces length at DB level |
| Soft delete consistency | ✅ | `deleted_at` pattern consistent; application layer must filter `WHERE deleted_at IS NULL` |
| TIMESTAMPTZ consistency | ✅ | All timestamp fields use `TIMESTAMPTZ` (timezone-aware UTC) — correct for a multi-region SaaS |
| FK index coverage | ✅ | All FKs should have indexes; data model specifies indexes per entity |
| NOT NULL constraints | ✅ | Required vs nullable fields specified in attribute tables |

### 5.3 GDPR Compliance Assessment

| Requirement | Status | Detail |
|-------------|--------|--------|
| PII identification | ✅ | `User.email` and `User.display_name` classified CONFIDENTIAL |
| Lawful basis | ✅ | Article 6(1)(b) — contract performance documented |
| Retention policy | ✅ | 90-day post-cancellation; hard deletion; ActivityLog 3-year hot + Glacier |
| Right to erasure | ✅ | Email → null; display_name → "Deleted User"; GDPR-compliant pseudonymisation |
| Data residency | ✅ | EU-West-2 (London); PostHog EU Cloud; SCCs for third-party processors |
| DPIA required | ✅ | Flagged; must be completed before production launch (GDPR Art. 35) |
| DPO appointment | ⚠️ | Required before production (GDPR Art. 37 if processing at scale); appointment pending |

---

## 6. Security Implementation Review

### 6.1 Authentication Implementation

| Control | Status | Detail |
|---------|--------|--------|
| Password hashing | ✅ | bcrypt, cost factor ≥ 12; `password_hash` VARCHAR(72) (bcrypt max) |
| JWT RS256 | ✅ | Asymmetric signing; private key in Secrets Manager; public key at JWKS endpoint |
| Access token lifetime | ✅ | 15-minute expiry; balanced security vs usability |
| Refresh token | ✅ | 7-day lifetime; HttpOnly cookie; rotated on use |
| Token revocation | ✅ | JWT blocklist in Valkey by `jti` claim with TTL |
| OAuth OIDC | ✅ | Google + Microsoft; PKCE enforced; state parameter for CSRF prevention |
| TOTP MFA | ⚠️ | `mfa_secret` AES-256 encrypted in DB; KMS envelope encryption specifics not documented — DLD-ADVISORY-04 |
| Email verification | ✅ | `email_verified` boolean; unverified users cannot access workspace features |
| Session invalidation | ✅ | Logout → add `jti` to Valkey blocklist; all active sessions revocable |

### 6.2 Authorization Implementation

| Control | Status | Detail |
|---------|--------|--------|
| RBAC roles | ✅ | admin / member / viewer; stored in `WorkspaceMember.role` |
| API middleware | ✅ | Fastify guard middleware checks `WorkspaceMember.role` per endpoint |
| DB-level role | ✅ | `api_user` restricted to CRUD on business tables; `audit_user` INSERT-only on `activity_log` |
| Cross-workspace isolation | ✅ | All queries scoped by `workspace_id`; FK relationships prevent cross-workspace data access |
| Permission matrix | ⚠️ | Role capability matrix (which endpoints admin/member/viewer can access) not produced as DLD artefact; required before Sprint 1 |
| IAM DB auth | ✅ | RDS IAM authentication (token-based); no static DB passwords |

### 6.3 Encryption and Secrets

| Control | Status | Detail |
|---------|--------|--------|
| Encryption at rest (RDS) | ✅ | AES-256 via AWS KMS; enabled at RDS instance level |
| Encryption at rest (ElastiCache) | ✅ | ElastiCache at-rest encryption enabled |
| Encryption in transit | ✅ | TLS 1.3 on all connections; HSTS headers; ALB HTTPS-only listener |
| Secrets management | ✅ | AWS Secrets Manager; 30-day rotation; no plaintext secrets in env vars or code |
| Field-level encryption (TOTP) | ⚠️ | AES-256 specified for `mfa_secret`; KMS key ID and envelope encryption procedure not documented |
| Key rotation | ✅ | Secrets Manager 30-day; KMS automatic annual key rotation |
| CORS | ✅ | Allowlist `https://app.quento.app`; `credentials: true` for SSE subscriptions |

### 6.4 Security Testing Plan

| Test Type | Status | Detail |
|-----------|--------|--------|
| SAST | ⚠️ | CodeQL referenced (GitHub Actions); not yet in pipeline configuration artefact |
| Dependency audit | ⚠️ | `npm audit` referenced; automated block threshold not defined |
| DAST (OWASP ZAP) | ❌ | Not yet planned — required for DLD-BLOCKING-04 (test strategy) |
| Penetration test | ❌ | Pre-production penetration test not scheduled; required before beta launch |
| Stripe webhook test | ⚠️ | Stripe CLI local webhook testing implied; CI pipeline integration not specified |

---

## 7. Integration Design Review

### 7.1 INT-001: OAuth OIDC (Google + Microsoft)

| Aspect | Status | Detail |
|--------|--------|--------|
| Protocol | ✅ | OIDC Authorization Code Flow with PKCE |
| Token exchange | ✅ | Server-side exchange; `id_token` validated; user profile `sub`, `email`, `name` extracted |
| New user provisioning | ✅ | Auto-create User record on first OAuth sign-in; link existing account if email matches |
| Error handling | ⚠️ | OAuth failure paths (user cancels, provider down, invalid state) not detailed |
| Timeout | ⚠️ | SLA dependency: auth flow < 2s p95; timeout budget for OIDC token endpoint not specified |

### 7.2 INT-002: Stripe (Billing)

| Aspect | Status | Detail |
|--------|--------|--------|
| Subscription lifecycle | ✅ | Create, update, cancel events handled via webhook |
| Webhook security | ✅ | `Stripe-Signature` HMAC validation; raw body preserved for signature verification |
| Idempotency | ⚠️ | Webhook handlers must be idempotent (Stripe can replay); idempotency key strategy not documented |
| Stripe Customer ID | ✅ | Stored in `Workspace.stripe_customer_id` |
| Error handling | ⚠️ | Stripe API timeout and retry policy not specified; circuit breaker not designed — DLD-ADVISORY-01 |
| Test mode | ⚠️ | Stripe test mode for CI/staging environment not documented |

### 7.3 INT-003: Email Delivery (Postmark / AWS SES)

| Aspect | Status | Detail |
|--------|--------|--------|
| Provider selection | ✅ | Postmark Year 1, AWS SES Year 2+ — migration path noted |
| Email types | ✅ | Task assignment, mention, invitation, password reset |
| Dispatch pattern | ❌ | Synchronous inline vs async queue pattern not specified — DLD-ADVISORY-05 |
| Retry policy | ⚠️ | Email delivery failure retry not specified; Postmark handles delivery retries but app-level handling needed |
| Template management | ⚠️ | Email templates (HTML + text) not referenced in design; Postmark template IDs or local templates not defined |
| PII handling | ✅ | Postmark DPA: 45-day retention; recipient email only; no content stored beyond delivery |

### 7.4 INT-004: PostHog Analytics (EU Cloud)

| Aspect | Status | Detail |
|--------|--------|--------|
| Data sent | ✅ | `distinct_id` = UUID (not email); anonymised events; `ph-no-capture` CSS class for PII fields |
| Session recording | ✅ | Enabled; PII fields masked via CSS |
| Feature flags | ✅ | Progressive rollout for new features; PostHog Node.js SDK on API Server |
| PECR consent | ⚠️ | Cookie consent gate for PostHog not yet implemented — HLD ADVISORY-05; required before production |
| Fire-and-forget | ✅ | Analytics is async; failure does not affect core API response path |
| Legal basis | ✅ | Legitimate interest; documented in data flow table |

---

## 8. Performance Design Review

### 8.1 NFR-P Coverage

| NFR | Target | Architecture Support | Status |
|-----|--------|---------------------|--------|
| NFR-P-001: API p95 | < 200ms | Fastify 76K req/s + Valkey task list cache (5-min TTL) + Kysely compiled queries | ✅ |
| NFR-P-002: LCP | < 2s | Next.js RSC (zero-JS server render) + CloudFront CDN for static assets | ✅ |
| NFR-P-003: SSE latency | < 500ms | Valkey pub/sub < 1ms + SSE write < 5ms; network RTT dominant | ✅ |
| NFR-P-004: DB query p95 | < 50ms | Indexed FKs + composite indexes + Kysely query cache | ✅ |

### 8.2 Caching Strategy

| Cache Key Pattern | TTL | Purpose |
|------------------|-----|---------|
| `tasks:workspace:{id}:list` | 5 minutes | Task list response caching; invalidated on task mutation |
| `JWT:BLOCKLIST:{jti}` | Remaining token lifetime | Revoked JWT tracking |
| `ratelimit:{type}:{id}:{window}` | Window duration | Sliding window rate limit counters |
| `sse:replay:{workspace_id}` | 10 minutes | Redis Streams missed-event replay buffer |
| Session state | 15 minutes (access token TTL) | Implicit in JWT lifetime; no server-side session store needed |

**Gap**: Cache invalidation strategy for task list cache on task mutation not specified. Invalidation patterns: (a) delete on write (`DEL tasks:workspace:{id}:list` on PATCH/DELETE task); (b) write-through; (c) TTL-only expiry. Option (a) is recommended.

### 8.3 Load Test Targets

| Scenario | Users | RPS | Duration | Success Criteria |
|---------|-------|-----|----------|-----------------|
| Normal load | 500 concurrent | 500 req/s | 30 min | p95 < 200ms, error rate < 0.1% |
| SSE fan-out | 5,000 SSE connections | — | 60 min | SSE delivery < 500ms, 0% disconnection |
| Peak load (3×) | 1,500 concurrent | 1,500 req/s | 15 min | p99 < 500ms, error rate < 1% |
| Database stress | 500 concurrent | DB-heavy | 15 min | DB p95 < 50ms, no deadlocks |

**Status**: ❌ Load test plan not yet documented — required as part of DLD-BLOCKING-04.

---

## 9. Operational Design Review

### 9.1 Observability Status

> ⚠️ **DLD-BLOCKING-02 (Inherited: HLD BLOCKING-03)**: Observability design is absent. The following assessment reflects the expected minimum viable observability design for a production SaaS service. These items are all required before Sprint 1.

| Area | Status | Required Specification |
|------|--------|----------------------|
| SLI definitions | ❌ | `request_latency_p95_ms`, `api_error_rate`, `sse_delivery_latency_ms`, `db_query_p95_ms`, `auth_failure_rate` |
| SLO definitions | ❌ | p95 < 200ms for 99.5% of requests in rolling 7-day window; error rate < 0.1%; uptime 99.9% |
| Error budget | ❌ | 43.2 min/month (0.1% of 30 days) for 99.9% SLO |
| Distributed tracing | ❌ | OpenTelemetry SDK for Fastify, Next.js, Kysely; trace context propagation; Grafana Tempo as backend |
| Structured logs | ❌ | JSON log schema: `{"timestamp":..., "level":..., "trace_id":..., "span_id":..., "service":..., "message":..., "context":{...}}` |
| Metrics | ❌ | Prometheus metrics via `@fastify/metrics`; Grafana Cloud Prometheus remote write |
| Alerting | ❌ | SLO burn-rate alerts (1-hour and 6-hour windows); PagerDuty or OpsGenie integration |
| Dashboards | ❌ | Service overview, error budget, SSE fan-out, DB performance dashboards |
| Runbooks | ❌ | Minimum: API high latency, DB failover, ElastiCache failover, ECS task failure |

### 9.2 Deployment Design

| Area | Status | Detail |
|------|--------|--------|
| CI/CD pipeline | ✅ | GitHub Actions; multi-environment (dev, staging, prod) |
| IaC | ✅ | OpenTofu; state in S3 with DynamoDB locking |
| Container registry | ⚠️ | ECR assumed; not explicitly specified in DLD artefacts |
| Deployment strategy | ⚠️ | ECS rolling update vs blue-green not specified — DLD-ADVISORY-03 |
| ECS task definition | ⚠️ | CPU/memory per task not specified — DLD-ADVISORY-03 |
| Health checks | ⚠️ | ALB health check path (`/api/health`) not defined in DLD — referenced in Section 3.2 |
| Rollback | ✅ | ECS rolling deploy with health check gate implies automatic rollback on failed health checks |
| Environment config | ✅ | SSM Parameter Store for static config; Secrets Manager for credentials |
| Secret injection | ✅ | ECS task role pulls from Secrets Manager at start; no plaintext in task definition |

### 9.3 Resilience Design

| Control | Status | Detail |
|---------|--------|--------|
| RDS Multi-AZ failover | ✅ | Automatic; < 60s DNS re-point; total RTO < 15 min (BLOCKING-01 resolved in HLD) |
| ECS task replacement | ✅ | ECS health check with replacement; ALB removes unhealthy targets automatically |
| ElastiCache Multi-AZ | ❌ | Single node Year 1 = SPOF — DLD-BLOCKING-01 |
| Circuit breakers | ❌ | No circuit breaker configuration for Stripe, Postmark/SES, PostHog — DLD-ADVISORY-01 |
| Retry policy | ❌ | Exponential backoff with jitter not specified for outbound calls |
| SSE reconnection | ✅ | `EventSource` auto-reconnect; Redis Streams replay |
| Timeout budgets | ⚠️ | Not specified per integration; Stripe < 3s p95, Postmark < 5s p95, PostHog async |

---

## 10. Testing Strategy Review

> ❌ **DLD-BLOCKING-04**: Testing strategy is absent from the DLD artefact set. The following section specifies the minimum required test strategy.

### 10.1 Required Test Strategy

| Layer | Framework | Coverage Target | Scope |
|-------|-----------|----------------|-------|
| Unit (API) | Vitest | ≥ 80% line coverage | Domain logic, validation rules, JWT helpers, rate limiting |
| Unit (Web) | Vitest + React Testing Library | ≥ 70% line coverage | Components, hooks, SSE client logic |
| Integration | Supertest + Testcontainers (PostgreSQL + Valkey) | Critical paths | Auth flows, task CRUD, workspace management, SSE pub/sub |
| E2E | Playwright | All 26 FR happy paths | Browser-level user journeys; WCAG 2.1 AA automated via `axe-playwright` |
| Performance | k6 | Scenarios in Section 8.3 | NFR-P-001 (p95 < 200ms), NFR-P-003 (SSE < 500ms), SSE fan-out |
| Security (SAST) | CodeQL (GitHub Actions) | All PRs | SQL injection, XSS, prototype pollution, dependency CVEs |
| Security (DAST) | OWASP ZAP | Pre-production | OWASP Top 10 scan against staging environment |
| Contract testing | Not specified | API schema validation | Validate OpenAPI spec conformance for all endpoints |

### 10.2 CI/CD Test Gates

| Stage | Tests Run | Gate |
|-------|-----------|------|
| PR → main | Unit + lint + SAST | Block merge if any test fails or critical CVE |
| main → staging | Unit + integration + e2e (subset) | Block deploy if error rate > 0% |
| staging → production | Full e2e + performance smoke | Block deploy if p95 > 200ms or error rate > 0.1% |
| Weekly | Full performance suite + DAST | Alert if regression detected |

---

## 11. HLD Conditions Verification

| HLD Condition | Resolution Status | DLD Action |
|--------------|------------------|------------|
| BLOCKING-01 (RTO < 15 min) | ✅ RESOLVED in HLD v1.1 | Confirmed: standard failover RTO < 15 min documented in DIAG-001 |
| BLOCKING-02 (ElastiCache SPOF) | ❌ NOT RESOLVED | Carried forward as DLD-BLOCKING-01; must specify Multi-AZ replication group |
| BLOCKING-03 (Observability absent) | ❌ NOT RESOLVED | Carried forward as DLD-BLOCKING-02; must design SLIs, SLOs, tracing, alerting, runbooks |
| ADVISORY-01 (Circuit breakers) | ❌ NOT ADDRESSED | Carried forward as DLD-ADVISORY-01 |
| ADVISORY-02 (WAF timeline) | ❌ NOT ADDRESSED | Carried forward as DLD-ADVISORY-02 |
| ADVISORY-03 (Full-text search Year 3) | ⚠️ Deferred | Acceptable to defer to Year 2 design phase |
| ADVISORY-04 (TOTP encryption specifics) | ❌ NOT ADDRESSED | Carried forward as DLD-ADVISORY-04 |
| ADVISORY-05 (PostHog PECR consent) | ❌ NOT ADDRESSED | Must be implemented before production (GDPR/PECR) |

---

## 12. Implementation Readiness Checklist

| Area | Ready? | Blocker |
|------|--------|---------|
| Technology stack finalised | ✅ | — |
| Data model complete and indexed | ✅ | — |
| Authentication flow specified | ✅ | — |
| RBAC matrix documented | ⚠️ | Permission matrix artefact missing |
| API endpoints defined | ⚠️ | OpenAPI spec not yet produced — DLD-BLOCKING-03 |
| SSE design complete | ✅ | Keepalive heartbeat interval to specify |
| Cache strategy defined | ⚠️ | Invalidation patterns for task list cache to document |
| Database migration strategy | ❌ | DLD-BLOCKING-05 |
| ElastiCache HA configured | ❌ | DLD-BLOCKING-01 |
| Observability designed | ❌ | DLD-BLOCKING-02 |
| Test strategy documented | ❌ | DLD-BLOCKING-04 |
| CI/CD pipeline specified | ⚠️ | GitHub Actions referenced; pipeline YAML not artefact |
| ECS task definitions | ⚠️ | CPU/memory per task not specified |
| Runbooks | ❌ | None produced — part of DLD-BLOCKING-02 |
| OpenAPI spec | ❌ | DLD-BLOCKING-03 |
| Pre-production pentest scheduled | ❌ | Required before beta |

---

## 13. Development Sequence Recommendations

Recommended sprint sequencing based on design dependencies:

```text
Sprint 1 (Foundations):
├── Resolve DLD-BLOCKING-01 (ElastiCache Multi-AZ config in OpenTofu)
├── Resolve DLD-BLOCKING-02 (Observability design + OpenTelemetry instrumentation)
├── Resolve DLD-BLOCKING-03 (Export and commit OpenAPI spec from @fastify/swagger)
├── Resolve DLD-BLOCKING-04 (Define test strategy, set up Vitest + Testcontainers)
└── Resolve DLD-BLOCKING-05 (Choose migration tool, create first baseline migration)

Sprint 2 (Auth + Workspace Core):
├── User registration + email verification (FR-001)
├── Email/password login + JWT issuance (FR-002)
├── OAuth OIDC integration (FR-001 + INT-001)
└── Workspace creation + WorkspaceMember management (FR-002 + FR-004)

Sprint 3 (Task Management Core):
├── Task CRUD + status transitions (FR-006 to FR-015)
├── Project management (FR-007)
├── Sub-task creation (FR-023 to FR-026)
└── ActivityLog write-through (FR for audit)

Sprint 4 (Real-Time + Notifications):
├── SSE endpoint + Valkey pub/sub fan-out (FR-003)
├── Notification entity + email dispatch (FR-005 + INT-003)
└── In-app notification read/unread (FR-005)

Sprint 5 (Billing + Admin):
├── Stripe subscription integration (INT-002)
├── Workspace admin panel (FR-004)
└── Billing lifecycle webhooks

Sprint 6 (Beta Hardening):
├── PostHog analytics + PECR consent gate (INT-004)
├── Performance test suite + optimisation
├── DAST scan (OWASP ZAP on staging)
└── Penetration test
```

---

## Appendix A: Blocking Issue Summary

| ID | Type | Priority | Description | Target |
|----|------|----------|-------------|--------|
| DLD-BLOCKING-01 | Inherited (HLD-B2) | Critical | ElastiCache single-node SPOF for security-critical functions | Before Sprint 1 |
| DLD-BLOCKING-02 | Inherited (HLD-B3) | Critical | Observability design absent (SLIs, SLOs, tracing, alerting, runbooks) | Before Sprint 1 |
| DLD-BLOCKING-03 | New | High | OpenAPI specification artefact not produced | Before Sprint 1 |
| DLD-BLOCKING-04 | New | High | Test strategy absent (pyramid, frameworks, coverage targets, performance plan) | Before Sprint 1 |
| DLD-BLOCKING-05 | New | High | Database migration strategy not specified (zero-downtime PostgreSQL) | Before Sprint 1 |

## Appendix B: Advisory Issue Summary

| ID | Type | Priority | Description |
|----|------|----------|-------------|
| DLD-ADVISORY-01 | Inherited (HLD-A1) | Medium | Circuit breaker configuration for Stripe, Postmark/SES, PostHog |
| DLD-ADVISORY-02 | Inherited (HLD-A2) | Medium | WAF timeline — deploy at launch or document compensating controls |
| DLD-ADVISORY-03 | New | Low | ECS task CPU/memory allocation not specified |
| DLD-ADVISORY-04 | Inherited (HLD-A4) | Medium | TOTP MFA secret KMS envelope encryption specifics |
| DLD-ADVISORY-05 | Inherited (HLD-A5) | High | PostHog PECR cookie consent gate — required before production |

## Appendix C: Requirements Coverage (DLD-Level)

| Requirement | Status | DLD Evidence |
|-------------|--------|-------------|
| FR-001: Authentication | ✅ | JWT RS256 + OAuth OIDC + bcrypt + email verification |
| FR-002: Workspace management | ✅ | Workspace + WorkspaceMember entities; Stripe integration |
| FR-003: Real-time collaboration | ✅ | SSE + Valkey pub/sub; EventSource reconnect; Streams replay |
| FR-004: RBAC | ✅ | admin/member/viewer; Fastify guard + DB role enforcement |
| FR-005: Notifications | ⚠️ | Entity defined; dispatch mechanism (sync vs async) pending |
| FR-006 to FR-022: Task management | ✅ | Task entity (status, priority, assignee, sub-tasks); API endpoints |
| FR-023 to FR-026: Sub-tasks | ✅ | Self-referential `parent_task_id`; depth = 1 at app layer |
| NFR-P-001: p95 < 200ms | ✅ | Fastify + Valkey cache + indexed DB queries |
| NFR-A-001: 99.9% uptime | ⚠️ | RDS Multi-AZ ✅; ElastiCache SPOF pending (DLD-BLOCKING-01) |
| NFR-A-002: RTO ≤ 30 min | ✅ | Standard failover < 15 min (HLD BLOCKING-01 resolved) |
| NFR-SEC-001: GDPR + data residency | ✅ | EU-West-2; PostHog EU; PII classified; DPIA flagged |
| NFR-SEC-002: Encryption at rest+transit | ✅ | AES-256 KMS + TLS 1.3 |
| INT-001 to INT-004 | ✅/⚠️ | OIDC ✅, Stripe ✅, Email ⚠️ (dispatch pending), PostHog ⚠️ (PECR pending) |
| DR-001 to DR-009 | ✅ | All 8 entities documented in ARC-001-DATA-v1.0 |

---

> **Generated by**: ArcKit `/arckit:dld-review` command
> **Generated on**: 2026-03-11 17:45 GMT
> **ArcKit Version**: 4.1.1
> **Project**: Task Management Portal (Project 001)
> **AI Model**: claude-sonnet-4-6
> **Generation Context**: ARC-001-HLDR-v1.1 (HLD review findings), ARC-001-DIAG-001-v1.0 (C4 Container + security architecture), ARC-001-DATA-v1.0 (8-entity data model), ARC-001-DFD-001-v1.0 (DFD + trust zones), ARC-001-REQ-v1.1 (FR/NFR/INT requirements), ARC-000-PRIN-v1.0 (architecture principles)