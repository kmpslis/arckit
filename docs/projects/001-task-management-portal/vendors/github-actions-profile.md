# Vendor Profile: GitHub Actions

**Slug**: github-actions
**Last Researched**: 2026-03-10
**Confidence**: High (5+ data points)

## Overview

GitHub Actions is GitHub's integrated CI/CD and automation platform, launched in 2019. It is deeply integrated with GitHub's version control, pull request, and security scanning features. As of 2025, it is the most widely used CI/CD platform for open-source projects and a dominant choice for UK SaaS startups that use GitHub for source control.

**Operated by**: GitHub (subsidiary of Microsoft since 2018)
**Launched**: 2019
**Marketplace actions**: 20,000+
**G2 Rating**: 4.5/5 (400+ reviews)

## Products and Services

| Feature | Description |
|---------|-------------|
| Workflow automation | YAML-based CI/CD pipelines triggered by events |
| Hosted runners | Ubuntu, Windows, macOS (Linux most common) |
| Self-hosted runners | Run on own infrastructure |
| Action marketplace | 20,000+ reusable workflow steps |
| GitHub Dependabot | Automated dependency vulnerability scanning and PRs |
| GitHub CodeQL | SAST static analysis (Advanced Security) |
| Secret scanning | Detect secrets committed to repo |
| OIDC federation | Keyless AWS/GCP/Azure auth from CI |
| Environments | Deployment protection rules, approvals |
| Artifacts / Cache | Build artifact storage and dependency caching |

## Pricing Model

As of mid-2025:

| Plan | Free Minutes | Storage | Price |
|------|-------------|---------|-------|
| Free (public repos) | Unlimited | 500MB | £0 |
| Free (private repos) | 2,000/month | 500MB | £0 |
| Team ($4/user/month) | 3,000/month | 2GB | ~£3.20/user/month |
| Enterprise | 50,000/month | 50GB | Contact sales |
| Additional minutes (Linux) | — | — | ~$0.008/min (~£0.006) |
| Additional storage | — | — | $0.25/GB/month |

Additional minute cost examples:
- 100 pipeline runs/day × 5 minutes each = 500 minutes/day = 15,000/month → overage ~£78/month (Team plan base = 3,000 free)

## UK Government Presence

- GitHub is listed on G-Cloud for source control services
- GDPR compliance via GitHub's DPA and SCCs
- GitHub Enterprise can be self-hosted (GitHub Enterprise Server) for data sovereignty

## Strengths

- Native GitHub integration — no separate authentication, webhook setup, or tool sprawl
- Dependabot provides automated dependency vulnerability scanning free (satisfies NFR-SEC-005)
- CodeQL SAST available free for public repos; included in GitHub Advanced Security for private
- OIDC federation with AWS — keyless, short-lived credential authentication (security best practice)
- 20,000+ marketplace actions cover every pipeline stage with battle-tested implementations
- Reusable workflows reduce pipeline code duplication across repositories
- GitHub Environments with protection rules enforce manual gates for production deployments
- Concurrent job matrix enables parallel test runs — faster feedback cycles

## Weaknesses

- Free private repo minutes (2,000/month) may be exceeded by teams with many daily builds
- Hosted runner performance is variable under high queue pressure
- Less sophisticated pipeline visualisation than CircleCI or Buildkite
- Self-hosted runners require maintenance and security patching
- GitHub vendor lock-in (though workflow YAML is portable to GitLab CI with adaptation)

## Projects Referenced In

- Task Management Portal (Project 001) — CI/CD pipeline recommendation (NFR-M-005)

## External Links

- Pricing: https://github.com/pricing
- Actions Marketplace: https://github.com/marketplace?type=actions
- OIDC AWS guide: https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services
