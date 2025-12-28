---
name: baryo-skills-lite
description: Condensed summaries of all 19 BaryoDev skills for token-efficient AI assistance (~2,500 tokens total vs 16,500 for full skills)
---

# BaryoDev Skills (Lite Edition)

**Usage**: Keep this in `.cursorrules`. For full details, say "Load baryo-X skill".

---

## 🔵 CORE SKILLS

### baryo-coding
**Library Mode**: Zero deps, `readonly struct`, `Span<T>`, no LINQ in hot paths, both sync+async APIs
**Application Mode**: Vertical slices, minimal Program.cs, DI container, feature folders
**Universal**: Nullable enabled, XML docs on public APIs, descriptive names, no exceptions for flow control

### baryo-testing
- Test: happy path + edge cases + failure modes
- Use fluent assertions: `result.Should().BeTrue()`
- BenchmarkDotNet for perf-critical code
- Categories: Unit, Integration, Benchmark

### baryo-discipline
- **Testing**: Never skip edge cases (null, empty, boundary, concurrency)
- **Anti-hallucination**: Never invent APIs, always ask for clarification
- **Deletion safety**: Scan usages, list impact, request permission before removing code
- **Documentation**: Create logs, report mistakes, ONLY propose skill updates (no auto-PR)

### baryo-learning
- Log every task: `.baryo/logs/YYYY-MM-DD-task.md`
- Report mistakes: `.baryo/mistakes/YYYY-MM-DD-mistake.md`
- Propose improvements: `.baryo/skill-proposals/`
- Create PRs to BaryoDev/skills ONLY after explicit manual user confirmation

---

## 🟢 PRODUCTION SKILLS

### baryo-security
- **Injection**: Parameterized queries, never concatenate user input
- **Auth**: bcrypt (12 rounds), JWT RS256, session rotation
- **Secrets**: Environment vars, never hardcode, rotate every 90 days
- **Headers**: CSP, HSTS, X-Frame-Options, X-Content-Type-Options
- **Validation**: Whitelist, not blacklist; validate all inputs

### baryo-observability
- **Logging**: Structured JSON, include traceId, never log PII
- **Levels**: ERROR (broken), WARN (unexpected), INFO (normal), DEBUG (troubleshooting)
- **Tracing**: OpenTelemetry, propagate context across services
- **Metrics**: RED method (Rate, Errors, Duration), expose /metrics
- **Health**: /health/live (app alive), /health/ready (deps available)

### baryo-compliance
- **Libraries**: MPL-2.0 license, file headers required
- **Demos/Templates**: MIT license
- **Git**: Sanitize history with git-filter-repo for sensitive data
- **Audit**: Run `npm audit` / `dotnet list package --vulnerable`

---

## 🟡 PROCESS SKILLS

### baryo-workflow
- **Sprint**: 2 weeks, Planning → Daily Standup → Review → Retro
- **Estimation**: 1-2-3-5-8 points, break down >8
- **Board**: Backlog → Sprint → In Progress → Review → Done
- **Definition of Done**: Code + Tests + Docs + Review + Deployed

### baryo-collaboration
- **PR Size**: <300 lines ideal, >500 must split
- **PR Title**: `<type>: <description>` (feat, fix, docs, perf)
- **Review**: Respond <24h, use [nit], [blocking], [question] prefixes
- **Merge**: Squash for features, require 1 approval + CI pass

### baryo-documentation
- **Tool**: VitePress for all docs
- **Structure**: guide/ (intro, install, quick-start), reference/ (changelog, faq, migration)
- **Changelog**: Keep a Changelog format, update BEFORE release
- **Style**: Active voice, real code examples, no "TODO" placeholders

### baryo-packaging
- **Versioning**: SemVer 2.0, manual version bumps
- **Metadata**: PackageId, Authors, Description, License, README
- **Release**: Clean → Build → Test → Pack → Verify → Push
- **Registry**: NuGet.org, npm, manual workflow_dispatch

---

## 🔴 ENTERPRISE SKILLS

### baryo-api
- **REST**: Nouns plural, resource-based, max 2 levels nesting
- **Response**: `{ data, meta, pagination }` or `{ error: { code, message, details } }`
- **Status**: 200 OK, 201 Created, 400 Bad Request, 401/403 Auth, 404 Not Found, 429 Rate Limited
- **Versioning**: URL path `/v1/`, support N-1, deprecation notices
- **Rate Limit**: Token bucket, return X-RateLimit-* headers

### baryo-privacy
- **Minimize**: Collect only necessary data, document purpose
- **PII**: Mark with @pii, never log, mask in errors
- **Consent**: Explicit, recorded with timestamp, withdrawable
- **Deletion**: GDPR Article 17, anonymize or delete, 30-day SLA
- **Export**: GDPR Article 20, provide user data in JSON/CSV

### baryo-scale
- **Caching**: Browser → CDN → Redis → DB, cache-aside pattern
- **Database**: Index frequently queried columns, avoid N+1, connection pools
- **Stateless**: No in-memory sessions, use Redis for shared state
- **Rate Limit**: Token bucket, distributed with Redis
- **Budget**: p95 <200ms, p99 <500ms, monitor and alert

### baryo-global
- **i18n**: Externalize all text, use Intl.DateTimeFormat/NumberFormat
- **RTL**: Use logical CSS properties (margin-inline-start)
- **a11y**: Semantic HTML, ARIA labels, keyboard navigation
- **Contrast**: 4.5:1 normal text, 3:1 large text (WCAG 2.1)

### baryo-devops
- **Pipeline**: lint → test → security → build → deploy
- **Docker**: Multi-stage builds, non-root user, minimal base image
- **IaC**: Terraform/Pulumi, version controlled, tag all resources
- **Deploy**: Blue-green or canary, zero-downtime, rollback tested

### baryo-resilience
- **Circuit Breaker**: Closed → Open → Half-Open, fail fast
- **Retry**: Exponential backoff + jitter, max 3 attempts
- **Backup**: 3-2-1 rule (3 copies, 2 types, 1 offsite)
- **DR**: RTO/RPO defined, tested quarterly, documented runbook

### baryo-finops
- **Tagging**: All resources tagged (team, env, cost-center)
- **Right-size**: Monitor CPU/memory usage, downsize underutilized
- **Savings**: Reserved instances for baseline, spot for flexible
- **Alerts**: Budget thresholds at 80% and 100%

### baryo-architecture
- **Structure**: Vertical slices by feature, not layers
- **Startup**: Fail fast on missing config, validate at boot
- **Observability**: Serilog + Prometheus + HealthChecks
- **DI**: Minimal Program.cs, feature-specific registration

---

## 📚 Full Skills

For complete details with examples: https://github.com/BaryoDev/skills

Say "Load baryo-X skill" for the full version of any skill above.
