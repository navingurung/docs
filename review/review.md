# 3C Code Review Guide

## 1. Framework

| C | Covers | Ask |
|---|---|---|
| **Correctness** | Functionality, Format, Security | Does it work? Is it safe? |
| **Clarity** | Readability, Structure | Can a human follow it? |
| **Complexity** | Maintainability, Performance | Will it scale / stay easy to change? |

Security lives under Correctness (broken if unsafe). Performance lives under Complexity (inefficient code = harder to maintain, slower to run).

## 2. Execution Workflow

**Step 1 — Automated run (before human review)**
- Linter (ESLint / Ruff) catches format issues
- CI tests catch functionality regressions
- If CI is red, bounce the PR back — don't review yet

**Step 2 — High-level pass (Clarity)**
- Read the PR top to bottom on GitHub
- If you can't understand the change within ~2 minutes, stop and ask the author before digging deeper

**Step 3 — Deep dive (line-by-line, 3C checklist)**
- Correctness & Security: edge cases covered? inputs validated? auth/permissions checked?
- Clarity: names obvious? nesting reasonable? does the diff match the PR description?
- Complexity & Performance: simpler way to do this? any loop/query that won't scale (N+1, etc.)?

## 3. PR Review Mechanics (as assigned reviewer)

1. **Check CI status** first — GitHub Actions or equivalent. Red CI = don't review yet.
2. **Read the diff on GitHub** for the high-level pass.
3. **Pull the branch locally** if you need to run it or the diff is too large to reason about in the browser:
   ```bash
   git fetch origin
   git checkout <branch-name>
   ```
4. **Run linters/tests locally** if not already covered by CI, or to double-check.
5. **Leave comments inline on GitHub**, then set review status:
   - `Approve` — good to merge
   - `Request changes` — blocker found
   - `Comment` — feedback, non-blocking

## 4. Comment Formula: Request / Reason / Resource

- **Request** — what needs to change
- **Reason** — why, tied back to Correctness / Clarity / Complexity
- **Resource** — a snippet or fix suggestion

> Example: "Please move this data fetch outside the `map` loop [Request]. Running queries inside a loop creates an N+1 performance bottleneck [Reason]. Fetch all IDs at once using `whereIn()` instead: `[snippet]` [Resource]."

## 5. Linter/Formatter Reference (set up later, during testing)

**Frontend (Next.js / TS)**
```bash
npm install -D eslint prettier eslint-config-prettier
npx eslint . --fix
npx prettier --write .
```

**Backend (FastAPI / Python)**
```bash
pip install ruff
ruff check . --fix
ruff format .
```

## 6. PR Review Checklist (quick reference)

- [ ] CI passing
- [ ] Correctness: logic sound, edge cases handled, no security gaps
- [ ] Clarity: readable, well-structured, matches PR description
- [ ] Complexity: no unnecessary complexity, no obvious perf issues
- [ ] Comments left in Request/Reason/Resource format where issues found
