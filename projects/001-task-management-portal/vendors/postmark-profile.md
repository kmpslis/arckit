# Vendor Profile: Postmark

**Slug**: postmark
**Last Researched**: 2026-03-10
**Confidence**: High (5+ data points)

## Overview

Postmark is a transactional email delivery service operated by ActiveCampaign (acquired 2022). It is specifically and exclusively designed for transactional email (receipts, notifications, password resets, invitations) — it deliberately does not support marketing/bulk email. This product focus is its primary competitive advantage: by maintaining separate sending infrastructure for transactional email only, Postmark achieves consistently high inbox placement rates.

**Founded**: 2009
**Acquired by ActiveCampaign**: 2022
**Headquarters**: Chicago, IL, USA
**G2 Rating**: 4.9/5 (500+ reviews)

## Products and Services

| Product | Description |
|---------|-------------|
| Transactional Email | REST API for sending individual transactional emails |
| Email Templates | Handlebars-based templating engine with visual editor |
| Inbound Email Processing | Parse and route inbound email to webhooks |
| Message Streams | Separate streams for transactional vs. broadcast (post-2021) |
| Bounce/Spam Management | Automated suppression lists; webhook notifications |
| Delivery Webhooks | Delivery, bounce, open, click, spam report events |
| DKIM/SPF | Included; custom domain signing |
| Dedicated IPs | Available on higher-volume plans |

## Pricing Model

As of mid-2025 (estimate — verify at postmarkapp.com/pricing):

| Plan | Monthly Emails | Price/Month |
|------|---------------|-------------|
| Free trial | 100 emails | £0 |
| Starter | 10,000 | ~£15 |
| Pro | 50,000 | ~£50 |
| Enterprise | 100,000+ | ~£100–150 |

- Overage: ~£1.25 per 1,000 emails above plan limit
- Annual billing: typically 10–20% discount

## UK Government Presence

- Not applicable (private sector SaaS product)
- Data processed in the US; Standard Contractual Clauses (SCCs) in place for GDPR compliance
- GDPR data processing addendum available

## Strengths

- Highest deliverability reputation in transactional email segment
- Median delivery time under 10 seconds (sub-second for many ISPs)
- Excellent developer experience — clean REST API, Node.js/Python/Ruby/PHP SDKs
- Delivery webhooks are reliable and well-documented
- 4.9/5 G2 rating — consistently highest-rated email service provider
- Deliberately limited to transactional email protects sending reputation
- Transparent deliverability stats in dashboard

## Weaknesses

- US-hosted infrastructure (not EU data residency); SCCs required for GDPR
- More expensive per email than AWS SES at high volume (> 500K emails/month)
- Acquired by ActiveCampaign — some concern about product direction post-acquisition
- No marketing email support (by design — may require separate vendor for marketing)
- Limited to email (no SMS, push, in-app notifications)

## Projects Referenced In

- Task Management Portal (Project 001) — Primary email delivery recommendation (with AWS SES as fallback)

## External Links

- Pricing: https://postmarkapp.com/pricing
- API Documentation: https://postmarkapp.com/developer
- GDPR: https://postmarkapp.com/eu-privacy-shield
