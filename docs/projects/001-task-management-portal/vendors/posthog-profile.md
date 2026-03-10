# Vendor Profile: PostHog

**Slug**: posthog
**Last Researched**: 2026-03-10
**Confidence**: High (5+ data points)

## Overview

PostHog is an open-source product analytics platform founded in 2020 (Y Combinator W20). It provides product analytics, session recording, feature flags, A/B testing, and user surveys in a single integrated platform. PostHog is unique in offering a fully open-source, self-hostable version alongside a managed cloud offering with an EU region — making it the leading GDPR-compliant product analytics choice for European SaaS companies.

**Founded**: 2020
**Headquarters**: San Francisco, CA / London, UK (distributed team)
**GitHub Stars**: ~25,000 (posthog/posthog)
**License**: MIT (open source core)
**G2 Rating**: 4.5/5 (400+ reviews)

## Products and Services

| Product | Description |
|---------|-------------|
| Product Analytics | Funnels, retention, paths, trends, cohorts |
| Session Recording | Full session replay with privacy masking |
| Feature Flags | Gradual rollouts, A/B testing, multivariate |
| Experiments (A/B) | Statistical significance testing |
| Surveys | NPS, CSAT, open-ended in-app surveys |
| Data Warehouse (beta) | Sync events to BigQuery/Snowflake |
| Heatmaps | Click, scroll, move heatmaps |
| Self-Hosting | Docker / Kubernetes deployment on own infra |

## Pricing Model

PostHog Cloud (as of mid-2025):

| Tier | Events/Month | Price |
|------|-------------|-------|
| Free | Up to 1,000,000 | £0 |
| Paid (events) | > 1M | ~£0.000248/event (~£248/1M) |
| Session recordings | > 5,000/month | ~£0.0045/recording |
| Feature flags | > 1M requests | ~£0.0001/request |

PostHog Self-Hosted: Free (hosting costs on own infrastructure only).

- EU Cloud region: All data stored in the EU (Frankfurt) — no cross-border transfer
- US Cloud region: Data in the US — requires SCCs for EU customers

## UK Government Presence

- Not applicable (private sector)
- EU Cloud region satisfies UK GDPR data residency without SCCs
- Data Processing Agreement (DPA) available

## Strengths

- EU region available — GDPR compliance without SCCs; simplest legal path
- Built-in `opt_in_capturing()` / `opt_out_capturing()` consent API — trivial to implement NFR-C-003
- Open source escape hatch — self-host if pricing becomes prohibitive or compliance requires it
- Feature flags and A/B testing included at no extra cost — significant product iteration value
- Session recording helps identify UX issues (NPS improvement, SD-7)
- 1M events/month free — Year 1 cost is £0 for most SaaS startups
- NPS surveys built-in — supports BR-004 (NPS 40+ target)
- Active development (frequent releases); strong community

## Weaknesses

- Less mature than Mixpanel or Amplitude for complex multi-touch attribution
- Self-hosted version requires operational expertise (ClickHouse + PostgreSQL + Redis stack)
- Session recording raises privacy concerns — must configure data masking for PII fields
- EU Cloud region has slightly fewer features than US Cloud region (historically)
- Smaller customer success team than Mixpanel/Amplitude at enterprise scale
- Data warehouse integrations still in beta as of 2025

## Projects Referenced In

- Task Management Portal (Project 001) — Analytics platform recommendation (INT-004)

## External Links

- Pricing: https://posthog.com/pricing
- GitHub: https://github.com/posthog/posthog
- EU Cloud: https://eu.posthog.com
- GDPR: https://posthog.com/docs/privacy/gdpr
