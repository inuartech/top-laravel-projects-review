/plan

Analyze all projects in this workspace and rank them from best to worst.f

Goal:
Produce a complete engineering portfolio review of all projects, including static analysis, architecture assessment, maintainability evaluation, engineering activity metrics, and comparative ranking.

For each project:

1. Static Analysis
- Run PHPStan from the repository root.
- Export results as JSON.

Command:

PHPSTAN_TABLE_ERROR_FORMATTER_FORCE_SHOW_ALL_ERRORS=1 ./vendor/bin/phpstan analyse --memory-limit=2G <project-folder> --error-format=json > <project-name>.json

- Count the total number of PHPStan issues.
- Categorize issues by severity if possible.
- Include issue density (issues per 1,000 lines of code if feasible).

2. Architecture Review
- Analyze overall architecture.
- Identify architectural strengths and weaknesses.
- Evaluate separation of concerns.
- Evaluate modularity and coupling.
- Identify signs of good domain modeling.

3. Code Quality Review
- Assess readability and consistency.
- Evaluate testing approach and test quality.
- Identify code smells.
- Assess naming conventions and maintainability.

4. Technical Debt & Risk
- Identify major technical debt.
- Highlight high-risk areas.
- Identify security concerns.
- Identify scalability concerns.
- Identify dependency risks.

5. Design Patterns
- Identify design patterns in use.
- Determine whether each pattern is implemented appropriately.
- Highlight anti-patterns or misuse of patterns.
- Score design pattern implementation quality.

6. Engineering Activity
- Analyze git history.
- Calculate commits per day over the last 90 days.
- Count merged PRs during the last 90 days.
- Count contributors during the last 90 days.
- Highlight signs of active or inactive maintenance.

Do not infer quality from framework choice or repository structure alone.
Support findings with concrete examples from the codebase, commit history, tests, and static analysis results.

Scoring

Score each project from 1-10 for:

- Architecture
- Code Quality
- Maintainability
- Design Patterns
- Technical Debt
- Engineering Activity

Calculate an overall score using:

- Architecture: 25%
- Code Quality: 25%
- Maintainability: 20%
- Design Patterns: 10%
- Technical Debt: 10%
- Engineering Activity: 10%

Final Output

Produce:

1. Executive summary of all projects.
2. Ranking table.

| Rank | Project | Overall Score | Architecture | Code Quality | Maintainability | Design Patterns | Technical Debt | Activity | PHPStan Issues |
|------|---------|---------------|--------------|--------------|----------------|----------------|---------------|----------|---------------|

3. Detailed scorecard for each project containing:
- Executive Summary
- Strengths
- Weaknesses
- Key Risks
- Architecture Review
- Code Quality Review
- Design Pattern Review
- Technical Debt Review
- Activity Metrics
- PHPStan Findings
- Recommendations

4. Conclude with:
- Best overall project
- Most maintainable project
- Best architecture
- Most active project
- Project requiring the most improvement

