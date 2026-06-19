# Analysis Methodology

> Engineering portfolio review of 11 open-source Laravel applications — June 2026

---

## Overview

Each project was evaluated across three independent pillars: **static analysis** (PHPStan), **architecture review** (code reading by AI agents), and **git activity metrics** (commit history). The three pillars feed into six scored dimensions, combined into a weighted overall score.

---

## 1. PHPStan Baseline

### Setup

A single root-level `composer.json` was used for all projects, containing only:

```json
{
  "require-dev": {
    "larastan/larastan": "^3.10"
  }
}
```

PHPStan was run against each project from the workspace root — without loading each project's own autoloader, bootstrap, or `phpstan.neon` config. This is a deliberate design choice for cross-project comparability (see Limitations).

### Command

```bash
PHPSTAN_TABLE_ERROR_FORMATTER_FORCE_SHOW_ALL_ERRORS=1 \
  /opt/homebrew/opt/php@8.4/bin/php ./vendor/bin/phpstan analyse \
  --memory-limit=2G apps/<project> \
  --error-format=json 2>/dev/null > outputs/<project>.json
```

Key flags:
- `--error-format=json` — machine-readable output for programmatic analysis
- `2>/dev/null` — suppresses PHPStan progress bars from contaminating JSON stdout
- `/opt/homebrew/opt/php@8.4/bin/php` — explicit PHP 8.4 binary (system default was 7.2, incompatible with larastan ^3.10)
- `--memory-limit=2G` — required for large codebases (invoiceninja at 1M LOC)
- Sequential execution — parallel runs caused OOM failures

### Output Parsing

```python
import json
d = json.load(open('outputs/<project>.json'))
total_errors = d['totals']['file_errors'] + d['totals']['errors']
```

### PHPStan Error Density

```
density = total_errors / (LOC / 1000)
```

Density is reported as errors per thousand lines of code (errors/KLOC). It normalises for codebase size, allowing direct comparison between a 5K-line package and a 1M-line application.

**Density interpretation:**

| Density (errors/KLOC) | Signal |
|-----------------------|--------|
| < 5 | Strong (or explicit PHP with few framework calls) |
| 5 – 20 | Moderate |
| 20 – 60 | Concerning |
| > 60 | Severe or package-bootstrap dependent |

See the caveat in §5 — density from a root scan is sensitive to how much a project relies on Laravel's global helpers vs explicit PHP.

---

## 2. Git Activity Metrics

All metrics use a 90-day window relative to the analysis date (June 2026).

```bash
# Commits in last 90 days
git -C apps/<project> log --oneline --since="90 days ago" | wc -l

# Unique contributor emails in last 90 days
git -C apps/<project> log --since="90 days ago" --format="%ae" | sort -u | wc -l

# Merge commits (proxy for merged PRs) in last 90 days
git -C apps/<project> log --since="90 days ago" --merges --oneline | wc -l
```

LOC counts were derived from `find apps/<project> -name "*.php" | xargs wc -l` excluding `vendor/`. Test file counts used `find apps/<project>/tests -name "*.php" | wc -l` (adapted per project: monica uses `tests/`, koel uses `tests/`, bagisto uses `packages/*/tests/`).

**Monica exception**: The git submodule was pinned at an August 2025 commit. No commits appeared in the 90-day window — not because the project is inactive (it is actively maintained upstream), but because the submodule was not updated. Activity was scored at 2/10 with this caveat noted.

---

## 3. Architecture Review

Each project was reviewed by an independent analysis agent that read (but did not execute) the following files:

- `composer.json` — dependency graph, Laravel version, autoload strategy
- Top-level directory listing — presence of `app/`, `src/`, `modules/`, `packages/`, `tests/`
- 2–3 representative service, repository, or model files chosen for complexity and centrality
- A sample test file
- `phpstan.neon` or `phpstan.neon.dist` (where present)
- `README.md`

**Project-specific adaptations:**
- **bagisto** — reviewed `packages/Webkul/` rather than `app/` (module-based architecture)
- **october** — reviewed `modules/` and `plugins/` rather than `app/` (CMS plugin architecture)
- **subscriptionify** — reviewed `src/` rather than `app/` (standalone package, no full app scaffold)
- **cachet** — reviewed available shell files; domain logic is in a private `cachethq/core` package inaccessible for review

---

## 4. Scoring Rubric

### Dimensions

Six dimensions are scored independently on a 1–10 integer scale.

#### Architecture (weight: 0.25)

Evaluates structural organisation, separation of concerns, and adherence to stated or implicit architectural intent.

| Score | Criteria |
|-------|----------|
| 9–10 | Clear architectural philosophy consistently applied. Domain boundaries enforced. No leaking abstractions. |
| 7–8 | Mostly consistent structure. Minor inconsistencies or partially applied patterns. |
| 5–6 | Mixed structure. Some areas well-organised, others ad hoc. |
| 3–4 | Ad hoc organisation. Flat structure, no enforced layering. |
| 1–2 | No discernible architecture. Everything in one namespace or layer. |

#### Code Quality (weight: 0.25)

Evaluates type safety, PHP idiom use, use of modern language features, and absence of anti-patterns at the expression level.

| Score | Criteria |
|-------|----------|
| 9–10 | Strict types throughout. Readonly/final classes where appropriate. No dynamic string magic. Targeted suppressions only. |
| 7–8 | Generally typed. Occasional dynamic access or legacy patterns. |
| 5–6 | Mixed typing. Some areas strictly typed, others loose. |
| 3–4 | Minimal typing. Relies on `mixed`, `array`, untyped properties. |
| 1–2 | No type discipline. Extensive dynamic access and magic methods. |

#### Maintainability (weight: 0.20)

Evaluates test coverage, documentation, onboarding clarity, and how easy it is for a new contributor to locate, understand, and change code.

| Score | Criteria |
|-------|----------|
| 9–10 | Test files mirror source structure. CLAUDE.md or equivalent enforces conventions. High test density on critical paths. |
| 7–8 | Good test coverage on happy paths. Some critical paths uncovered. |
| 5–6 | Tests exist but are sparse or concentrated on non-critical code. |
| 3–4 | Minimal tests. Hard to onboard without tribal knowledge. |
| 1–2 | No tests. No documentation beyond README. |

#### Design Patterns (weight: 0.10)

Evaluates correct application of known patterns (Repository, Service Object, Action, DTO/Value Object, Strategy, Pipeline, Observer) and absence of anti-patterns (God class, global function bags, marker interfaces).

| Score | Criteria |
|-------|----------|
| 9–10 | Multiple patterns applied correctly and consistently. Anti-patterns absent. |
| 7–8 | Good pattern adoption. One or two anti-patterns present. |
| 5–6 | Partial pattern adoption. God classes or global helpers visible. |
| 3–4 | Anti-patterns dominant. Patterns present only in isolated areas. |
| 1–2 | No recognisable patterns. Procedural or controller-heavy. |

#### Technical Debt (weight: 0.10)

Evaluates known vulnerabilities, security advisory suppression, suppressed static analysis categories, vendor forking, and dev-main/unversioned dependencies.

| Score | Criteria |
|-------|----------|
| 9–10 | No known advisories. PHPStan at high level, no blanket suppressions. No forked vendor code. |
| 7–8 | Minor debt. PHPStan suppressions are targeted and commented. |
| 5–6 | Moderate debt. Some blanket suppressions or unaddressed advisories. |
| 3–4 | Significant debt. Suppressed advisories, forked vendor code, or unresolved error categories. |
| 1–2 | Critical debt. Security advisories suppressed in sensitive domain, dev-main dependencies, no static analysis. |

#### Engineering Activity (weight: 0.10)

Evaluates community velocity and bus-factor risk.

| Score | Commits/90d | Contributors/90d |
|-------|------------|-----------------|
| 10 | > 500 | > 20 |
| 8–9 | 200 – 499 | 10 – 20 |
| 6–7 | 50 – 199 | 5 – 10 |
| 4–5 | 10 – 49 | 2 – 5 |
| 1–3 | < 10 | ≤ 1 |

### Overall Score Formula

```
Overall = (Architecture × 0.25)
        + (Code Quality  × 0.25)
        + (Maintainability × 0.20)
        + (Design Patterns × 0.10)
        + (Technical Debt  × 0.10)
        + (Activity        × 0.10)
```

Scores are not rounded to the nearest integer — the weighted formula produces one decimal place that reflects genuine variation across dimensions.

---

## 5. Limitations and Caveats

### PHPStan root scan is not equivalent to project-level enforcement

Running PHPStan from the workspace root without each project's own bootstrap means:

- **Laravel facade calls resolve as errors.** `DB::select()`, `Cache::get()`, `Log::info()` are unresolvable without the project's autoloader. Projects that use facades heavily (coolify, october) show high error counts that partially reflect architecture style rather than actual bugs.
- **Global helper functions are unresolvable.** Functions defined in files loaded via `composer.json autoload.files` (coolify's `bootstrap/helpers/shared.php`, akaunting's `helpers.php`) generate `function.notFound` errors. coolify's 11,521 `function.notFound` errors are a real architectural signal (18 global helper files is a real problem) but the count is amplified by the scan setup.
- **Conversely, projects with explicit PHP get near-zero errors.** invoiceninja (2 errors) and akaunting (1 error) write explicit PHP that avoids facade magic. Their root-scan scores are underestimates of real issues — invoiceninja's `phpstan.neon` suppresses `Call to an undefined method` entirely, which would not show in a root scan regardless.

**This is a deliberate trade-off.** The root scan provides consistent comparability across all 11 projects. Per-project configs with project-specific bootstraps would be more accurate but incomparable, since each project suppresses different error categories at different levels.

### Architecture scores are based on sampled files

No analysis reviewed every file in a codebase. Architecture agents sampled representative files. For large projects (invoiceninja at 1M LOC, bagisto at 484K LOC), entire subsystems may differ from the sampled areas.

### PHPStan density is a proxy, not a quality score

Density (errors/KLOC from root scan) measures how much a project relies on Laravel's global helpers and facade magic vs explicit PHP. subscriptionify scores 119.0 errors/KLOC — the highest density in the corpus — yet ranks second overall. The errors are `function.notFound` for illuminate/support calls in a package that cannot bootstrap a full Laravel application. Density in this context measures **framework magic reliance**, not code defects.

### Git activity reflects the local submodule state

All git metrics read from the checked-out submodule, not the upstream remote. If a submodule is pinned to an old commit (as monica was, at August 2025), the 90-day window may show no activity even for an actively maintained upstream project.

### No runtime or integration testing

This analysis is entirely static — no application was run, no database was queried, no HTTP request was made. Runtime bugs, performance issues, and integration failures are outside scope.

---

## 6. Toolchain Versions

| Tool | Version |
|------|---------|
| PHP | 8.4 (Homebrew) |
| PHPStan | 2.2.2 |
| larastan/larastan | ^3.10 |
| Analysis date | June 2026 |
| Git activity window | 90 days (March 2026 – June 2026) |

---

*See [README.md](README.md) for the full ranking table and key findings. See [INSIGHTS.md](INSIGHTS.md) for editorial synthesis.*
