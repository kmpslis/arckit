# Vendor Profile: Grafana Cloud

**Slug**: grafana-cloud
**Last Researched**: 2026-03-10
**Confidence**: High (5+ data points)

## Overview

Grafana Cloud is the managed cloud offering of Grafana Labs, built on the open-source Grafana stack: Grafana (dashboards), Loki (logs), Prometheus (metrics), Tempo (distributed traces), and Mimir (long-term metrics storage). Grafana Labs was founded in 2014; Grafana Cloud launched in 2020. It is the leading open-source observability stack and a strong alternative to Datadog for cost-conscious teams.

**Founded**: 2014 (Grafana Labs)
**Headquarters**: New York, NY / Stockholm, Sweden (distributed)
**GitHub Stars (grafana/grafana)**: ~65,000
**G2 Rating**: 4.4/5 (700+ reviews)

## Products and Services

| Product | Description |
|---------|-------------|
| Grafana (dashboards) | Industry-standard visualisation and alerting |
| Loki (logs) | Log aggregation — label-based querying |
| Prometheus / Mimir (metrics) | Time-series metrics with PromQL |
| Tempo (traces) | Distributed tracing — OTLP and Jaeger compatible |
| k6 Cloud | Load testing as a service |
| Faro (RUM) | Real User Monitoring for frontend |
| OnCall | On-call scheduling and escalation |
| SLO module | Service Level Objective tracking |

## Pricing Model

Grafana Cloud (as of mid-2025):

| Tier | Included | Price |
|------|---------|-------|
| Free | 50GB logs, 10K active metric series, 50GB traces, 14-day retention | £0 |
| Pro | Above + usage-based | ~£50/mo base |
| Usage-based overages | Logs: ~£0.50/GB; Metrics: ~£8/1K series; Traces: ~£0.50/GB | Per consumption |
| EU region | Included | No extra charge |

## UK Government Presence

- Not specifically targeted at UK Gov; no G-Cloud listing
- EU region (Frankfurt) satisfies GDPR data residency for operational telemetry
- DPA available for GDPR compliance

## Strengths

- Generous free tier covers all Year 1 observability needs for a small SaaS startup
- Native OpenTelemetry support across Loki, Tempo, and Prometheus — no proprietary agents
- Prometheus-compatible metrics format (explicitly required by NFR-M-001)
- EU region — operational telemetry kept in EU for GDPR compliance
- Pre-built dashboards for Node.js, PostgreSQL, Redis, Kubernetes available in Grafana marketplace
- SLO module enables SLO-based alerting (NFR-M-001: SLO-based alerts)
- Open source escape hatch — all components (Grafana, Loki, Prometheus, Tempo) are self-hostable
- k6 Cloud integration for load testing (NFR-M-005 pipeline stage)

## Weaknesses

- Free tier log retention is 14 days (need to configure long-term S3 archival for 1-year audit log requirement — NFR-C-002)
- Loki query language (LogQL) has a steeper learning curve than Elasticsearch/Kibana for teams accustomed to ELK
- Less mature APM (code-level profiling) compared to Datadog — no automatic instrumentation
- Grafana OnCall (on-call scheduling) is less feature-rich than PagerDuty
- Dashboards require manual configuration (no auto-discovery like Datadog)

## Projects Referenced In

- Task Management Portal (Project 001) — Observability/APM recommendation (NFR-M-001)

## External Links

- Pricing: https://grafana.com/pricing/
- GitHub: https://github.com/grafana/grafana
- OpenTelemetry docs: https://grafana.com/docs/opentelemetry/
