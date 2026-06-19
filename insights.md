# Engineering Insights: Open-Source Laravel Portfolio Review

> Analysed: 11 projects · June 2026 · PHPStan 2.2.2 · PHP 8.4 · Larastan 3.x

---

## 1. Cross-Project Failure Patterns

Five failure modes appear across nearly every project regardless of size, activity level, or team:

### 1.1 PHPStan Used as Theatre, Not Enforcement

Nine of eleven projects have PHPStan in their toolchain. Only two use it honestly.

- **subscriptionify** runs at level `max` with larastan — zero blanket suppressions
- **koel** runs at level 5 with targeted, commented ignores
- **invoiceninja** suppresses `'Call to an undefined method'` entirely — in a financial application handling payments and tax calculations. This is the most alarming single configuration decision in the corpus.
- **akaunting** has no PHPStan config at all, despite processing invoices, reconciliations, and multi-currency transactions
- **coolify** and **bookstack** both run at level 4 with meaningful suppressions

The pattern: PHPStan gets added to `require-dev`, a config is written at whatever level passes without effort, and the results are never surfaced in CI in a way that blocks merges. The tool becomes a checkbox rather than a gate.

**Evidence**: `apps/invoiceninja/phpstan.neon:36-50` — blanket suppression of six error categories including method-not-found. `apps/akaunting/` — no phpstan.neon exists.

### 1.2 God Classes Are Universal — Only the Name Changed

The "fat controller" died. The God class lived on in new containers:

| Project | God Class | LOC |
|---------|-----------|-----|
| coolify | `app/Jobs/ApplicationDeploymentJob.php` | 4,894 |
| invoiceninja | `app/Http/Controllers/Portal/PdfBuilder.php` | 2,410 |
| bagisto | `packages/Webkul/DataTransfer/src/Helpers/Importers/Product/Importer.php` | 2,377 |
| bagisto | `packages/Webkul/Checkout/src/Cart.php` | 1,204 |
| laraowl | `app/Services/RecordService.php` | 977 |
| invoiceninja | `app/Repositories/BaseRepository.php::alternativeSave()` | 444 lines in one method |

The adoption of Action classes, job queues, and service objects did not solve God class proliferation — it moved the blob from controllers into jobs and importers. **coolify's ApplicationDeploymentJob is the most instructive case**: the project uses lorisleiva/laravel-actions correctly everywhere *except* its most important operation, where all the complexity landed in one 4,894-line file that handles Git, Docker, Nixpacks, health checks, rollback, and multi-server deployment simultaneously.

### 1.3 Global Function Bags Are the New Global Variables

Every project except koel, subscriptionify, and monica relies on large files of globally autoloaded helper functions:

- **coolify**: 18 helper files loaded via `bootstrap/helpers/`, totalling 8,000+ LOC. `shared.php` alone is 4,176 lines.
- **invoiceninja**: Three global helper files (`TranslationHelper.php`, `Generic.php`, `ClientPortal.php`) via `composer.json autoload.files`
- **akaunting**: `app/Utilities/helpers.php` at 483 lines containing `user()`, `company()`, `setting()`, `role_model_class()`

These functions cannot be type-checked, mocked in tests, or replaced via the service container. They are effectively global variables with function syntax. The projects that avoided this pattern (koel, subscriptionify, monica) all score in the top three.

### 1.4 Security Advisory Suppression in Financial Applications

This is the most serious finding in the corpus:

- **akaunting** (`apps/akaunting/composer.json`): 22 Packagist security advisories deliberately suppressed with `block-insecure: false`. Akaunting is an accounting application handling invoices, taxes, bank reconciliation, and financial reporting. Leaving 22 known vulnerabilities unaddressed is not a maintenance convenience — it is a liability.
- **october** (`apps/october/composer.json`): `"advisories": {"block": false}` — security advisory blocking explicitly disabled.

The pattern in both cases appears to be "it blocked our install and we needed to ship" — a short-term fix that becomes a permanent policy.

### 1.5 Test Coverage Imbalance: Strong Where Visible, Absent Where Critical

Every project has a showcase test path. Almost none have comprehensive coverage of their highest-risk code:

- **bagisto**: 11 of 42 packages tested. The untested packages include Product, Sales, Checkout, Tax, Shipping, and Inventory — the entire commerce core.
- **akaunting**: 38 test files for 240K LOC (~1% ratio) for a financial application
- **cachet**: Zero test files anywhere
- **coolify**: 407 test files with excellent security tests — but `ApplicationDeploymentJob` (the most critical, most complex class) has no dedicated test coverage because branching paths in a 4,894-line class are nearly impossible to enumerate

---

## 2. Architectural Tendencies

What patterns are converging across the Laravel ecosystem, and which projects are ahead of the curve?

### Leading patterns (in adoption order)

**Vertical slice / feature folder organisation** is the most significant architectural shift visible across the corpus. Monica is the clearest example: `app/Domains/{Domain}/{Feature}/` contains every service, controller, viewhelper, and test for that feature in one place. Koel's namespace mirrors this. This is a clear departure from the flat Laravel scaffold (Controllers/, Models/, Services/ at the top level).

**Action classes replacing fat controllers** are now mainstream. Coolify uses lorisleiva/laravel-actions. Laraowl has an Actions/ directory. Koel uses service objects with focused responsibilities. This is now the ecosystem norm, not the exception.

**readonly final Value Objects / DTOs** are emerging as the mark of architecturally serious projects. Subscriptionify's `ConsumptionResult`, `FeatureInfo`, `ResolvedFeature` — all `final readonly` classes with `private __construct` and `static make()`. Koel's entire `app/Values/` directory follows the same pattern. Monica partially uses this. Most other projects still pass associative arrays.

**AI integration as a first-class concern**: Three projects have dedicated AI namespaces already — koel (`app/Ai/` with Agents, Tools, Services, Serializers), coolify (laravel/mcp + laravel/ai), bagisto (laravel/ai 0.2.2). This is not experimental; it's core product.

**Repository pattern is not dying** — it's table stakes. Every medium-to-large project has repositories. Koel shows what mature repositories look like: interface-backed, read-only, with custom Eloquent builders handling query complexity.

### Still stuck in 2018

- Akaunting's `override/` vendor forking approach — patching Illuminate and livewire source directly
- Bagisto's empty marker interfaces (`interface Product {}`) — the appearance of contract programming without enforcement
- Invoiceninja's 93,972-line `SMSNumbers.php` PHP array file — should be a lazy-loaded data source
- String-based event names (`'catalog.product.create.after'`) instead of typed Event classes

---

## 3. Developer Contribution Analysis

### Prolific contributors (all-time commit depth)

| Developer | Email | Project | All-time Commits | Characterisation |
|-----------|-------|---------|----------------:|-----------------|
| David Bomba | turbo124@gmail.com | invoiceninja | 18,552 | Depth: solo founder, built 1M LOC |
| Andras Bacsai | andras.bacsai@gmail.com | coolify | ~11,200 | Depth: most active open-source infra project |
| Samuel Georges | sam@daftspunk.com | october | 8,441 | Depth: October CMS co-founder/maintainer |
| Dan Brown | ssddanbrown@googlemail.com | bookstack | 4,118 | Depth: solo maintainer, high architectural discipline |
| Phan An | me@phanan.net | koel | 3,156 | Depth: highest code quality in corpus |

### Breadth contributors (appear across multiple projects)

No contributor appears across multiple projects in this corpus — each project has a distinct, independent team. This is expected for unrelated open-source applications, but it also means no cross-pollination of architectural patterns between teams.

### Commit quality signals

**Phan An (koel)** — Follows Conventional Commits rigorously (`feat:`, `fix:`, `chore:`, `refactor:`). Commit messages are specific and descriptive. Commits are small and focused. The CLAUDE.md file (which encodes architecture rules for AI agents and contributors) was written by him — an unusual and valuable piece of documentation that no other project in this corpus has.

**Dan Brown (bookstack)** — Consistent, high-quality commits over years. The codebase reads as one author with an unwavering vision. The `ssddanbrown/asserthtml` package (used in tests) was authored by him, signalling genuine investment in testing infrastructure rather than just test count.

**Andras Bacsai (coolify)** — Very high velocity (898 commits in 90 days, 60 contributors). Commit messages are consistent. The security test suite (CommandInjectionSecurityTest) shows a clear security mindset. The main signal of scaling pain is the ApplicationDeploymentJob.

**David Bomba (invoiceninja)** — 18,552 commits is a singular achievement. The codebase shows signs of the weight: BaseRepository.alternativeSave() (444 lines), suppressed PHPStan categories, and inherited debt from building a product this large over many years. The architectural instincts (repository layer, fractal transformers, payment driver strategy) are sound — the execution shows the cost of sustained solo velocity.

---

## 4. Who Is Worth Following

These five individuals, based on code quality signals, commit discipline, and architectural choices, are worth tracking:

### 1. Phan An — `@phanan` — koel

**Why**: Highest-scoring project in the corpus (8.70). More importantly, the *architecture is documented as enforceable rules* — `CLAUDE.md` prohibits `config()` calls inside services (must use `#[Config]` attribute), requires repositories to be read-only, mandates `createOne()`/`createMany()` in factories. These are decisions other developers write in Slack and forget; he turned them into a document that applies to AI agents and humans equally.

**What to learn**: The Saloon HTTP integration pattern, custom Eloquent builders per model, readonly Value Objects with private constructors, and the Pipeline pattern for multi-step API enrichment chains. Also: how to choose PHPStan suppressions (small, targeted, commented with reason) rather than blanket category blocks.

**Commit signals**: Small, focused, Conventional Commits. Koel's 248 commits in 90 days average ~14 files per commit (estimated). Zero merged PRs means direct push to main — confident in automated gates.

### 2. Dan Brown — `@ssddanbrown` — bookstack

**Why**: Building and maintaining a 189K LOC wiki platform solo, at 8/8/8/8/7 across all dimensions, is the strongest individual performance in this corpus. The Queries/Repos split (separate read-side query objects from write-side repositories) is an uncommonly mature architectural decision. He wrote his own HTML DOM assertion library for tests. This is not someone following best practices — this is someone who arrived at them independently.

**What to learn**: Domain-driven module layout (`app/Entities/`, `app/Permissions/`, `app/Search/`), the Query Object pattern for read-side Eloquent, JointPermission pre-computed permission table as a performance optimisation, and how to build a productive solo open-source project.

### 3. Alexis Saettler — monica team

**Why**: Monica's `BaseService` + `ServiceInterface` pattern is the cleanest implementation of the Service Object pattern in the corpus. Every one of the 261 service classes goes through the same lifecycle: `rules()` → `permissions()` → `execute()`. The `QueuableService` extension lets any service become a dispatchable job with zero code duplication. 516 test files, each named and structured to mirror `app/Domains/`. The maintainability score (9/10) reflects a project where future contributors can locate any code in under 30 seconds.

**What to learn**: Vertical-slice domain architecture, the `BaseService` lifecycle pattern, `QueuableService` for async services, and the discipline of mirroring test structure to source structure.

### 4. Andras Bacsai — `@andrasbacsai` — coolify

**Why**: Managing 898 commits across 60 contributors in 90 days while producing `ValidationPatterns.php` (a centralised regex constants class with factory methods for security-critical rules) and a security test suite that catches shell injection, IDOR, and path traversal attacks shows genuine engineering leadership at scale. Most people running a project this fast would skip the security tests entirely.

**What to learn**: How to write security regression tests (parameterised Pest datasets for attack payloads), how to use `lorisleiva/laravel-actions` correctly, and how to wire OpenAPI 3.0 attributes directly on controllers to keep documentation in sync.

### 5. James Brooks — `@jbrooksuk` — cachet

**Why**: A counterintuitive inclusion. Cachet is the weakest project in the corpus overall, but James Brooks built the original Cachet status page application — and the architecture of the new version (a thin shell delegating to a private package with well-structured config) shows someone who understood "this codebase was getting too large, so I extracted the domain into a proper package." The *intent* is correct even if the execution (no tests, dev-main dependency) is not production-ready.

---

## 5. Which Project Is Worth Contributing To

**For learning modern Laravel patterns: koel**
Koel has the clearest contribution rules (CLAUDE.md), the strictest automated gates, and a maintainer who values code quality over velocity. A PR to koel will be reviewed against explicit architectural standards, which is better feedback than "looks good." The codebase is small enough (72K LOC) that a new contributor can understand the whole system. Start with the `tests/` directory — 323 files to learn from.

**For maximum community impact: coolify**
898 commits, 60 contributors, 360 merged PRs in 90 days. The project is genuinely used in production by thousands of self-hosters. A contribution here matters in a tangible way. The tradeoff: the codebase is noisier and the feedback loop is faster but less precise. Avoid `ApplicationDeploymentJob.php` — contribute to the Actions/ layer instead.

**For domain expertise in financial PHP: invoiceninja**
The largest open-source PHP billing system. If your interest is financial domain modelling (payment gateways, recurring billing, tax rules, multi-currency), there is nowhere better. Caveats: PHPStan suppressions and 93K-line data arrays are warning signs of accumulated debt.

**To avoid as a contribution target:**
- **cachet** — near-zero activity, no tests, dev-main core dependency
- **bagisto's Cart.php** — 1,204 LOC God class with no tests, any contribution risks breakage
- **coolify's ApplicationDeploymentJob.php** — contributions here are high-risk, low-reviewability

---

## 6. Red Flags

### Cachet
The entire application shell is 35 PHP files. All domain logic is in a private `cachethq/core` package pinned to `dev-main` with no git hash, no version tag, and no CI pipeline to detect when it breaks. There are zero test files. Two commits in 90 days from one contributor. This is not a production-ready engineering reference — it is a prototype.

### Akaunting's Security Culture
22 Packagist security advisories suppressed with `block-insecure: false` in `composer.json`. Vendor code forked in `overrides/` (Illuminate, Livewire, maatwebsite/excel, symfony/process) with no plan to upstream. A financial application with these characteristics is, by definition, a liability. The engineering patterns (Job lifecycle, Events, Scopes) are instructive; the security practices should not be emulated.

### Coolify's ApplicationDeploymentJob
Even in an otherwise well-engineered project, this 4,894-line file is a lesson in what happens when complexity is accepted rather than decomposed. It handles Git clone, Nixpacks build, Railpack build, Dockerfile build, Docker Compose, health checks, rollback, and multi-server deployment in one class. Adding a new deployment target requires understanding the entire file. This is precisely the scenario the Action pattern was supposed to prevent.

### PHPStan Suppression as a Pattern
Any project that suppresses `'Call to an undefined method'` across the entire codebase (invoiceninja) or runs with `block-insecure: false` (october, akaunting) is signalling one thing: the toolchain was adopted to satisfy a checklist, not to enforce quality. These suppressions represent unquantified technical debt that accumulates silently.

---

## 7. Lessons Learned

### 1. A CLAUDE.md that encodes architecture as rules is more valuable than a wiki

Koel's CLAUDE.md explicitly states: "Repositories are for *reading* data only — never write data in a repository." "Never call `config()` directly — use the `#[Config]` attribute." "Use `throw_if`/`throw_unless` instead of `if (...) throw`." These are actionable, checkable rules. Every other project in the corpus has either no documentation or prose documentation that describes what the code does, not what it must do. The CLAUDE.md approach makes architecture refusals automatic rather than opinion-based.

### 2. Static analysis level matters more than its presence

PHPStan at level 4 with broad suppressions is marginally better than no PHPStan. The false confidence it provides may actually be worse than nothing, because it signals "we have static analysis" while hiding the same bugs it would catch at level 6. The projects with the fewest bugs visible in code review (koel, subscriptionify, monica) all run at higher effective levels or with genuine larastan integration.

### 3. The Action pattern doesn't solve the God class problem — it moves it

The universal adoption of Action classes (coolify, laraowl, koel) is a genuine improvement over fat controllers. But every project with complex operations eventually produces a God class somewhere — just not in the controller anymore. Coolify's ApplicationDeploymentJob, bagisto's Cart.php, and invoiceninja's PdfBuilder all demonstrate this. The lesson: decomposition requires discipline at every level of the stack, not a single pattern.

### 4. Package constraints force better API design

Subscriptionify, a 5K LOC standalone package, scores higher than every full application except koel and monica. The constraints of package development (you cannot know what the host application looks like; your public API is permanent; your models must be swappable) force decisions that application developers avoid: interface contracts, injectable dependencies, config-driven behaviour, and testability in isolation. Building a package before a feature is a useful forcing function.

### 5. PHPStan error density from a root scan is a proxy for implicit PHP, not quality

Our baseline PHPStan scan (no project configs, no autoloaders) generates wildly different error counts: invoiceninja gets 2, akaunting gets 1, but coolify gets 22,065 and subscriptionify gets 593. This does not mean invoiceninja is 11,000× better than coolify. It means invoiceninja's code is more explicit (fewer unresolved facade calls, fewer Laravel magic methods) while coolify's architecture relies more on globally-autoloaded helper functions that PHPStan cannot resolve without the project's own bootstrap. Error density in this context measures **how much the project relies on Laravel's magic vs explicit PHP** — a real signal, but not the whole picture.

---

*Analysis performed June 2026. Git activity window: last 90 days from analysis date. PHPStan run from workspace root without project-specific configs (consistent baseline). Architecture scores based on direct code review of representative files.*
