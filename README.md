# Laravel Framework Analysis

A comparative engineering review of 11 production open-source Laravel applications, conducted in June 2026 using static analysis, architecture code review, and git activity metrics.

**Deliverables**

| File | Description |
|------|-------------|
| [`INSIGHTS.md`](INSIGHTS.md) | Editorial synthesis — patterns, red flags, who to follow, lessons learned |
| [`METHODOLOGY.md`](METHODOLOGY.md) | Full methodology: scoring rubric, PHPStan baseline, git metrics, limitations |
| [`outputs/index.html`](outputs/index.html) · [Live](https://claude.ai/code/artifact/42562f7f-8024-4f0a-9349-93ce768f5ae8) | Interactive scorecard dashboard with rankings, scorecards, insights, contributor profiles |
| `outputs/<project>.json` | Raw PHPStan JSON output per project |

---

## Projects

All 11 projects live as git submodules under `apps/`.

| Project | Domain | Laravel | LOC | Test Files | Maintainer |
|---------|--------|---------|-----|-----------|------------|
| [akaunting](apps/akaunting) | Accounting | 12 | 240K | 38 | akaunting team |
| [bagisto](apps/bagisto) | E-commerce | 12 | 484K | 2 | Webkul |
| [bookstack](apps/bookstack) | Wiki platform | 12 | 189K | 161 | Dan Brown |
| [cachet](apps/cachet) | Status page | 12 | 2.5K | 0 | James Brooks |
| [coolify](apps/coolify) | Self-hosted PaaS | 12 | 228K | 407 | Andras Bacsai |
| [invoiceninja](apps/invoiceninja) | Invoicing SaaS | 12 | 1.04M | 442 | David Bomba |
| [koel](apps/koel) | Music streaming | 11 | 72K | 323 | Phan An |
| [laraowl](apps/laraowl) | Uptime monitoring | 12 | 12K | 19 | laraowl team |
| [monica](apps/monica) | Personal CRM | 11 | 134K | 516 | Alexis Saettler |
| [october](apps/october) | CMS | 11 | 187K | 1 | Samuel Georges |
| [subscriptionify](apps/subscriptionify) | Subscription package | 12 | 5K | 12 | Small team |

---

## Rankings

Scored across six weighted dimensions. See [METHODOLOGY.md](METHODOLOGY.md) for the full rubric and weighting formula.

| Rank | Project | Overall | Arch | Quality | Maint | Patterns | Debt | Activity | PHPStan Errors | PHPStan /KLOC | Commits/90d |
|------|---------|--------:|-----:|--------:|------:|---------:|-----:|---------:|---------------:|--------------:|------------:|
| 1 | **koel** | **8.70** | 9 | 9 | 9 | 9 | 8 | 7 | 6,059 | 83.8 | 248 |
| 2 | **subscriptionify** | **8.20** | 9 | 9 | 8 | 9 | 8 | 4 | 593 | 119.0 | 19 |
| 3 | **monica** | **7.95** | 9 | 8 | 9 | 9 | 8 | 2 | 6,657 | 49.5 | 0† |
| 4 | **bookstack** | **7.70** | 8 | 8 | 8 | 8 | 7 | 6 | 6,153 | 32.6 | 111 |
| 5 | **october** | **7.15** | 8 | 7 | 7 | 8 | 6 | 6 | 11,789 | 63.0 | 122 |
| 6 | **invoiceninja** | **6.85** | 7 | 6 | 7 | 7 | 5 | 10 | 2 | 0.0 | 657 |
| 7 | **bagisto** | **6.80** | 7 | 7 | 6 | 7 | 6 | 8 | 14,067 | 29.1 | 326 |
| 8 | **coolify** | **6.65** | 7 | 6 | 7 | 6 | 4 | 10 | 22,065 | 97.0 | 898 |
| 9 | **laraowl** | **6.40** | 7 | 7 | 6 | 7 | 6 | 4 | 786 | 63.6 | 21 |
| 10 | **akaunting** | **6.25** | 7 | 6 | 6 | 7 | 5 | 6 | 1 | 0.0 | 143 |
| 11 | **cachet** | **4.95** | 6 | 7 | 3 | 6 | 4 | 1 | 48 | 19.0 | 2 |

† monica: git submodule pinned at Aug 2025 — zero commits in the 90-day window. Activity scored conservatively at 2/10.

---

## Key Findings

**Architecture gold standard** — koel is the only project where architecture is encoded as machine-checkable rules (CLAUDE.md prohibits `config()` calls in services, requires read-only repositories, mandates typed factory methods). Every other project documents architecture in prose or not at all.

**Most dangerous single configuration** — akaunting suppresses 22 Packagist security advisories with `block-insecure: false` in a financial application handling invoices, reconciliation, and multi-currency transactions.

**Worst single class** — coolify's `ApplicationDeploymentJob.php` at 4,894 LOC handles Git, Docker, Nixpacks, Railpack, health checks, rollback, and multi-server deployment simultaneously. The project uses the Action pattern correctly everywhere else.

**PHPStan as theatre** — 9 of 11 projects have PHPStan in `require-dev`. Only 2 (koel, subscriptionify) use it at a level that catches real bugs. invoiceninja suppresses `Call to an undefined method` across the entire codebase.

**Live security vulnerability** — laraowl's `VerifyLaraowlToken` middleware compares API tokens with direct string equality instead of `hash_equals()`, making it vulnerable to timing attacks.

For the full analysis, read [`INSIGHTS.md`](INSIGHTS.md) or open the [interactive dashboard](https://inuartech.com/laravel-projects-reviewed/).

---

## Toolchain

```
PHPStan 2.2.2 · larastan/larastan ^3.10 · PHP 8.4
Analysis date: June 2026
Git activity window: 90 days prior to analysis date
```

See [METHODOLOGY.md](METHODOLOGY.md) for full details on how scores were derived.
