 Engineering Portfolio Review — Laravel Framework Analysis

 Context

 The workspace at /Users/nei/src/idhl/laravel-framework-analisy contains 11 open-source Laravel projects under apps/ as git submodules, collected for comparative analysis. The goal is to produce a complete engineering
 portfolio review: static analysis, architecture assessment, code quality, technical debt, design patterns, and engineering activity — then rank each project and publish an HTML artifact.

 Root toolchain: ./vendor/bin/phpstan (larastan ^3.10) is installed and ready to use. project-b.json at the root contains plain-text PHPStan table output (not JSON) from a previous interrupted run — can be ignored.

 All 11 submodules are fully checked out. All are valid git repos tracked under .git/modules/apps/<project>.

 ---
 Projects Under Review

 All paths are relative to /Users/nei/src/idhl/laravel-framework-analisy/apps/.

 ┌─────────────────┬───────────┬──────────────────────────┬────────────┬─────────────┬──────────────────┬───────────────────┬────────────────────────────────────────────────┐
 │     Project     │ PHP Files │ Total LOC (excl. vendor) │ Test Files │ 90d Commits │ 90d Contributors │  PHPStan Config   │                     Notes                      │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ akaunting       │ 3,830     │ 240K                     │ 38         │ 143         │ 5                │ none              │ Full accounting app                            │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ bagisto         │ 2,572     │ 484K                     │ 2          │ 326         │ 15               │ none              │ Ecommerce, module-based (packages/Webkul/)     │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ bookstack       │ 1,758     │ 189K                     │ 161        │ 111         │ 6                │ phpstan.neon.dist │ Wiki platform                                  │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ cachet          │ 35        │ 2.5K                     │ 0          │ 2           │ 1                │ none              │ Status page (nearly dormant)                   │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ coolify         │ 1,798     │ 228K                     │ 407        │ 898         │ 60               │ none              │ PaaS/self-hosted                               │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ invoiceninja    │ 4,030     │ 1.04M                    │ 442        │ 657         │ 11               │ phpstan.neon      │ Invoicing SaaS (largest codebase)              │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ koel            │ 1,395     │ 72K                      │ 323        │ 248         │ 6                │ phpstan.neon.dist │ Music streaming                                │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ laraowl         │ 157       │ 12K                      │ 19         │ 21          │ 5                │ none              │ Small monitoring app                           │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ monica          │ 1,649     │ 134K                     │ 516        │ 0           │ 0                │ phpstan.neon      │ CRM (0 local commits — shallow clone)          │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ october         │ 1,558     │ 187K                     │ 1          │ 122         │ 10               │ none              │ October CMS (modules/plugins/themes structure) │
 ├─────────────────┼───────────┼──────────────────────────┼────────────┼─────────────┼──────────────────┼───────────────────┼────────────────────────────────────────────────┤
 │ subscriptionify │ 68        │ 5K                       │ 12         │ 19          │ 3                │ phpstan.neon      │ Standalone Laravel package (src/ not app/)     │
 └─────────────────┴───────────┴──────────────────────────┴────────────┴─────────────┴──────────────────┴───────────────────┴────────────────────────────────────────────────┘

 ---
 Execution Plan

 Phase 1: PHPStan Static Analysis

 Run the exact command specified for each project from the workspace root (/Users/nei/src/idhl/laravel-framework-analisy):

 PHPSTAN_TABLE_ERROR_FORMATTER_FORCE_SHOW_ALL_ERRORS=1 \
   ./vendor/bin/phpstan analyse --memory-limit=2G apps/<project> \
   --error-format=json > <project>.json

 Projects → output files (all written to workspace root):
 - apps/akaunting → akaunting.json
 - apps/bagisto → bagisto.json
 - apps/bookstack → bookstack.json
 - apps/cachet → cachet.json
 - apps/coolify → coolify.json
 - apps/invoiceninja → invoiceninja.json
 - apps/koel → koel.json
 - apps/laraowl → laraowl.json
 - apps/monica → monica.json
 - apps/october → october.json
 - apps/subscriptionify → subscriptionify.json

 Important constraints:
 - Run one project at a time to avoid OOM (--memory-limit=2G per run).
 - The root phpstan runs without each project's own autoloader; this produces more false positives (unresolved facades, missing model types) but gives consistent cross-project comparability.
 - After each run, parse the JSON: totals.file_errors + totals.errors = total issues. Calculate issue density = total_issues / (total_loc / 1000).
 - For bagisto, the main code is in apps/bagisto/packages/Webkul/; the scan still targets apps/bagisto which includes that path.

 Phase 2: Metrics Collection (Bash, no agents)

 For each project collect via git commands (all 11 dirs are under apps/<project>):
 # Commits last 90 days
 git -C apps/<project> log --oneline --since="90 days ago" | wc -l

 # Unique contributors last 90 days
 git -C apps/<project> log --since="90 days ago" --format="%ae" | sort -u | wc -l

 # Merged PRs (merge commits last 90 days)
 git -C apps/<project> log --since="90 days ago" --merges --oneline | wc -l

 # Commits per day (90d window)
 echo "scale=2; <commit_count> / 90" | bc

 LOC and test counts are already collected (see table above). For monica, verify: git -C apps/monica log --oneline | head -10 — the repo is valid but the shallow clone has no recent-window commits; note this in the
 report and score conservatively.

 Phase 3: Architecture & Code Quality Analysis (Parallel Agents)

 Launch one general-purpose agent per project in parallel (11 agents total). Each agent receives:
 - The project name, full path (/Users/nei/src/idhl/laravel-framework-analisy/apps/<project>), and known stats
 - Instructions to READ (not execute) key files:
   - composer.json — dependencies, Laravel version
   - app/ directory listing — architectural layers present
   - 2–3 representative service/repository/model files
   - A sample test file
   - Any phpstan.neon or phpstan.neon.dist
   - README.md for context

 Each agent returns a structured assessment covering:
 1. Architecture score (1–10) + justification with file citations
 2. Code quality score (1–10) + justification
 3. Maintainability score (1–10) + justification
 4. Design patterns score (1–10) + identified patterns and anti-patterns
 5. Technical debt score (1–10) + key risk areas
 6. Strengths (3 bullet points with file citations)
 7. Weaknesses (3 bullet points with file citations)
 8. Key risks (2 bullet points)
 9. Specific recommendations (3 actionable items)

 Phase 4: Engineering Activity Score

 Scoring rubric for Engineering Activity (1–10):
 - 10: >500 commits/90d, >20 contributors
 - 8–9: 200–499 commits/90d, 10–20 contributors
 - 6–7: 50–199 commits/90d, 5–10 contributors
 - 4–5: 10–49 commits/90d, 2–5 contributors
 - 1–3: <10 commits/90d or zero activity

 Phase 5: PHPStan Issue Scoring

 Technical debt score is influenced by PHPStan but also by the manual review. Scoring guidance:
 - Issue density < 1 per KLOC: strong
 - 1–5 per KLOC: moderate
 - 5–20 per KLOC: concerning
 ▎ 20 per KLOC: severe

 Phase 6: Overall Score Calculation

 Overall = (Architecture × 0.25) + (Code Quality × 0.25) +
           (Maintainability × 0.20) + (Design Patterns × 0.10) +
           (Technical Debt × 0.10) + (Engineering Activity × 0.10)

 Phase 7: Insights Generation (insights.md)

 Write /Users/nei/src/idhl/laravel-framework-analisy/insights.md — a synthesis document aimed at a senior developer deciding which projects or people to follow for learning. This is not a scorecard; it is editorial and
 opinionated.

 Sections to include:

 1. Cross-Project Failure Patterns
 What do nearly all 11 projects get wrong? Look for shared blind spots across PHPStan findings, test coverage, architectural anti-patterns, and dependency hygiene. Cite at least 3 concrete tendencies with examples from
 the codebase.

 2. Architectural Tendencies
     from the codebase.

     2. Architectural Tendencies
     What patterns are converging across the Laravel ecosystem? (e.g. Livewire adoption, Action classes replacing fat controllers, shift away from repositories, use of DTOs/Value Objects.) Which projects are ahead of
     the curve vs still using 2018-era patterns? Cite specific files/directories as evidence.

     3. Developer Contribution Analysis
     Using git log data (--format="%ae %s") across all projects:
     - Identify the most prolific contributors by email/name across all 11 repos
     - Identify "breadth" contributors (appear in multiple projects)
     - Identify "depth" contributors (high commit count in single project)
     - Flag any maintainers who show consistent code quality signals (e.g. writes tests, uses typed PHP, small focused commits)
     - Note commit message quality as a proxy for engineering discipline

     4. Who Is Worth Following
     Based on contributor analysis + code quality signals, name the top 3–5 individual contributors (by email alias or GitHub handle derived from commit metadata) and explain why — what specifically about their commits,
     patterns, or consistency makes them worth learning from.

     5. Which Project Is Worth Contributing To
     Honest take: given activity, code quality, and community size, which project would give the best return for a senior dev's open-source contribution time? Which would expose them to the most modern Laravel patterns?

     6. Red Flags
     Projects or patterns a senior dev should actively avoid as learning material — and why (e.g. bad coupling, no tests, outdated patterns, bus-factor risk).

     7. Lessons Learned
     3–5 distilled engineering lessons from this corpus, grounded in specific evidence from the analysis.

     The tone should be direct and evidence-based. Avoid generic advice. Every claim should be anchored to a file path, commit, or PHPStan result.

     ---
     Phase 8: HTML Artifact Generation

     Load the artifact-design skill and produce a polished, self-contained HTML file at:
     /private/tmp/claude-501/-Users-nei-src-idhl-laravel-framework-analisy/90f50d7c-aa50-4c9d-a4c4-a557ad181416/scratchpad/portfolio-review.html

     Content structure:
     1. Executive summary (2–3 paragraphs)
     2. Ranking table with all scores and PHPStan issue counts
     3. Detailed scorecard per project (accordion or tabs):
       - Executive summary, strengths, weaknesses, key risks
       - Architecture review
       - Code quality review
       - Design pattern review
       - Technical debt review
       - Activity metrics
       - PHPStan findings summary
       - Recommendations
     4. Key insights callout panel (summary of the most valuable findings from insights.md)
     5. Conclusion: best overall, most maintainable, best architecture, most active, most in need of improvement

     ---
     Special Project Notes

     - bagisto: Ecommerce app using a package/module structure. Most code lives in apps/bagisto/packages/Webkul/ not app/. PHPStan scans apps/bagisto which includes that path. Architecture agents should focus on
     packages/Webkul/.
     - cachet: Near-dormant (35 PHP files, 0 tests, 2 commits in 90d). Likely the weakest overall project. Score accordingly.
     - monica: Shows 0 commits in the local 90-day git window — shallow clone. Has the most test files (516) of all projects. Score activity conservatively with the caveat noted.
     - october: October CMS uses modules/ and plugins/ instead of standard app/. Architecture agents must look at apps/october/modules/ not just app/.
     - subscriptionify: A standalone Laravel package (no full app). Architecture scored against package-quality standards. Code lives in src/ not app/.
     - invoiceninja: Largest codebase (1M LOC). PHPStan run will take longest — allow up to 15 min. Has its own phpstan.neon.
     - bookstack/koel/invoiceninja/monica/subscriptionify: All have their own PHPStan config — architecture agents should read these to understand what the maintainers themselves enforce.

     ---
     Verification

     After each PHPStan run:
     python3 -c "import json; d=json.load(open('<project>.json')); print(d['totals'])"

     Final deliverables:
     - /Users/nei/src/idhl/laravel-framework-analisy/insights.md — editorial synthesis for senior devs
     - portfolio-review.html — deployed as a Claude Artifact with full scorecards

     ---
     Execution Order

     1. Run PHPStan for all 11 projects sequentially, largest-first:
     invoiceninja → bagisto → akaunting → coolify → bookstack → october → monica → koel → laraowl → subscriptionify → cachet
     2. Collect git metrics (commits, contributors, PRs, commits/day) and contributor email lists per project
     3. Launch 11 parallel architecture analysis agents
     4. Calculate weighted scores
     5. Write insights.md at workspace root (cross-project synthesis, contributor analysis, who to follow)
     6. Generate and deploy portfolio-review.html as a Claude Artifact (includes insights summary panel)

