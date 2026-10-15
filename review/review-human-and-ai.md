# Human + AI Code Review Guide

Companion to `code-review-guide.md` (3C Code Review Guide).
Use AI to **speed up** each step. The human reviewer **decides and owns** the approval.

> AI finds candidates. The human confirms, decides, and signs off.

---

## 0. Ground Rules

| Rule | Why |
|---|---|
| **AI never approves.** Only the human clicks Approve / Request changes. | Ownership stays with the reviewer. |
| **Verify every AI finding** against the actual code before commenting. | AI can be confidently wrong (false positives, invented APIs). |
| **Never paste secrets** — `.env`, API keys, tokens, passwords. | Security. |
| **Never paste real customer data** — passport info, names, card data, production DB dumps. Use masked or dummy data. | Privacy / compliance. |
| **Use only company-approved AI tools** for company code. | Code confidentiality. |
| **Rewrite AI output in your own words** before posting comments. | Clear, accountable feedback. |
| If AI and your judgment disagree, **your judgment wins** — or ask a teammate. | AI lacks full project context. |

---

## 1. Who Does What

| Step | Human | AI |
|---|---|---|
| 1. Understand Linear issue | Reads issue, confirms goal | Summarizes, lists ambiguities (Prompt 1) |
| 2. Understand PR description | Reads PR, checks it matches | Compares issue ↔ PR ↔ diff (Prompt 2) |
| 3. CI check | Checks CI status | — |
| 4. High-level pass | Reads diff | Explains the change if unclear (Prompt 3) |
| 5. Deep dive (3C) | Line-by-line review | Flags candidate issues (Prompt 4, 5) |
| 6. What-if testing | **Runs** the tests | Generates test cases (Prompt 6) |
| — Verify AI findings | Confirms / rejects each one | Self-checks findings (Prompt 7) |
| 7. Decide & notify | Sets GitHub status, sends Chatwork | Drafts Chatwork message (Prompt 8) |
| 8. Re-review | Confirms fixes | Checks fixes vs previous comments (Prompt 9) |

---

## 2. Getting the Inputs

Collect these before prompting. Use the GitHub CLI or copy from GitHub.

```bash
# PR title + description
gh pr view <PR_NUMBER> --json title,body

# PR diff
gh pr diff <PR_NUMBER>

# Or with plain git
git fetch origin
git diff origin/main...origin/<branch-name>
```

Linear issue: copy the title, description, and acceptance criteria.

> For very large diffs, review per file or per folder instead of pasting everything at once.

---

## 3. Prompts

Replace `{…}` with real content. Run **Prompt 0 first** in each new AI session.

---

### Prompt 0 — Context Setup (run once per session)

```
You are assisting a human code reviewer. You do NOT approve or reject PRs.
Your job is to find candidate issues and explain your reasoning. The human verifies everything.

Project: SAMURAI TAX — tax-free refund processing system for foreign tourists in Japan.
Stack:
- Frontend: Next.js, React, TypeScript, Tailwind CSS, shadcn/ui, MUI
- Backend: FastAPI (Python), Node.js
- DevOps: Docker, Docker Compose
- Code comments are written in Japanese (follow existing conventions).

Review framework (3C):
- Correctness: functionality, format, security
- Clarity: readability, structure
- Complexity: maintainability, performance

Rules:
- Only report issues you can point to in the given code (file + line or snippet).
- If you are unsure, say "uncertain" and explain what you would need to confirm.
- Do not invent functions, libraries, or APIs that are not in the code.
- Be concise. No praise padding.

Reply "Ready" and wait for the next input.
```

---

### Prompt 1 — Understand the Linear Issue

```
Here is the Linear issue:

{paste Linear issue title, description, acceptance criteria}

Please give me:
1. Goal — one sentence.
2. Acceptance criteria — as a checklist (rewrite clearly if vague).
3. Scope — what is in, what is out.
4. Affected areas — frontend / backend / admin / mobile / DB / external API.
5. Ambiguities — anything unclear, missing, or contradictory. Phrase each as a question I can ask the author.
6. Risks — what could go wrong with this kind of change.
```

👉 If item 5 has real gaps → use **Template A** from the main guide.

---

### Prompt 2 — Issue ↔ PR ↔ Diff Match

```
Linear issue:
{paste issue}

PR description:
{paste PR title + body}

Diff:
{paste diff}

Check and report:
1. Does the PR description explain what, why, and how to test? List anything missing.
2. Does the diff satisfy each acceptance criterion? Mark each: ✅ covered / ⚠️ partial / ❌ missing — with the file/line as evidence.
3. Unrelated changes — anything in the diff not explained by the issue or PR description.
4. Hidden changes — migrations, env vars, dependency changes, config changes, API response shape changes.
```

👉 Missing/mismatched → **Template A**.

---

### Prompt 3 — Explain the Change (when not understandable in ~2 min)

```
I can't follow this diff quickly. Please explain:

{paste diff}

1. Summary — what changed, in 3–5 bullet points.
2. Flow — how data moves through the changed code (request → logic → response / UI).
3. Key files — which files matter most, and which are minor.
4. Suggested review order — which file to read first.
5. Is this PR doing more than one thing? If yes, suggest how it could be split.
```

👉 Still unclear or should be split → **Template C**.

---

### Prompt 4 — 3C Review

```
Review this diff using the 3C framework.

Linear acceptance criteria:
{paste criteria}

Diff:
{paste diff}

For each issue, use this exact format:

[severity] file:line
- Request: what should change
- Reason: why (tag as Correctness / Clarity / Complexity)
- Resource: suggested fix or snippet
- Confidence: high / medium / low

Severity labels:
- [must] bug, security issue, or breaks acceptance criteria
- [should] strongly recommended
- [suggestion] optional improvement
- [nit] tiny style point
- [question] need clarification

Check especially:
- Correctness: edge cases, null/undefined, error handling, input validation, auth/permissions
- Clarity: naming, nesting, dead code, debug logs, consistency with existing conventions
- Complexity: N+1 queries, unnecessary re-renders, needless dependencies, hard-coded values, over-engineering

Group results by severity, [must] first. If there are no issues in a category, say "None found".
```

---

### Prompt 5 — Security & Data Check

Use for backend, auth, payment, refund, passport, or any user-data change.

```
Do a security-focused review of this diff:

{paste diff}

Check for:
1. Missing authentication or authorization (can another role / store / tenant access this?)
2. Missing input validation on API boundaries
3. Injection risks (SQL, command, XSS)
4. Secrets or tokens hard-coded or logged
5. Personal data (passport, name, address, payment) logged, exposed in responses, or stored unnecessarily
6. Unsafe error messages leaking internals
7. Missing rate limiting / idempotency on sensitive actions (refunds, payments, submissions)
8. CORS, cookie, or token handling issues

Report each finding with: file:line, risk, how it could be exploited, suggested fix, confidence.
If nothing is found, say so — do not invent issues.
```

---

### Prompt 6 — What-If Test Cases

```
Generate test cases for this change.

Linear issue:
{paste issue}

Diff:
{paste diff}

Create test cases for each relevant category. Skip categories that don't apply and say why.

Categories:
- Happy path
- Boundary values (empty, null, 0, negative, max length, large numbers)
- Input format (full-width/全角, emoji, special chars, wrong type)
- Invalid input (missing fields, malformed data)
- Backward compatibility (old mobile app versions, existing DB records, old API clients)
- Data migration (existing data, rollback)
- Regression (related features)
- Permissions (other role, other store, logged out, expired token)
- Concurrency (double submit, simultaneous edits, duplicate requests)
- Failure (timeout, external API/POS down, partial failure, retry duplicates)
- Date/time (JST vs UTC, midnight, month end, year end)
- Money/numbers (rounding, tax calculation, precision)
- UI states (loading, empty, error, long text, mobile width)
- Environment (new env vars, Docker vs local)
- Rollback (what breaks if this PR is reverted)

Output as a table:
| # | Category | What if… | Steps | Expected result | Priority (high/med/low) |

Put high-priority cases first.
```

👉 **You run the tests.** AI only writes the list.

---

### Prompt 7 — Verify AI Findings (reduce false positives)

Run after Prompt 4 / 5, in the same session.

```
Re-check your findings above. For each one:
1. Quote the exact code that proves the issue.
2. Is it a real issue, or could it be handled elsewhere (middleware, schema, parent component, DB constraint)?
3. Final verdict: keep / drop / needs human check — with one-line reason.

Be strict. Dropping a weak finding is better than keeping a false one.
```

👉 Then **you** open each kept finding in the code and confirm it yourself.

---

### Prompt 8 — Draft the Chatwork Message

```
Draft a Chatwork message to the PR author using this template:

Template: {A / B / C / D / E / F / G / H — paste the template from code-review-guide.md}

Details:
- Author name: {name}
- Linear ID: {LINEAR-ID}
- PR title: {title}
- PR URL: {url}
- Confirmed [must] issues: {list, with PR comment links}
- Confirmed [suggestion]/[nit]: {list}
- Tests I ran: {list}

Language: {Japanese / English}
Tone: polite, professional, short. Keep the Chatwork tags ([To:], [info], [title]) exactly as in the template.
```

---

### Prompt 9 — Re-review

```
This is a re-review.

My previous review comments:
{paste your earlier comments}

New commits since my last review:
{paste diff of new commits — GitHub "Changes since your last review", or git diff <old-commit>..<new-commit>}

For each previous comment:
- ✅ Fixed — show the code that fixes it
- ⚠️ Partially fixed — what's still missing
- ❌ Not fixed

Then check: did the new commits introduce any NEW issues? Use the same [severity] format.
```

👉 All fixed → **Template H**. Still issues → **Template D**.

---

## 4. Quick Flow (copy this)

```
[ ] Prompt 0 — context setup
[ ] Prompt 1 — understand Linear issue          → human confirms
[ ] Prompt 2 — issue ↔ PR ↔ diff match          → human confirms
[ ] CI green                                     → human checks
[ ] Prompt 3 — explain change (only if needed)
[ ] Prompt 4 — 3C review
[ ] Prompt 5 — security (if sensitive area)
[ ] Prompt 7 — AI self-verifies findings
[ ] Human verifies each finding in the code
[ ] Prompt 6 — what-if test cases               → human RUNS tests
[ ] Human writes GitHub comments (own words)
[ ] Human sets GitHub status
[ ] Prompt 8 — draft Chatwork message           → human edits & sends
[ ] Prompt 9 — re-review (after fixes)
```

---

## 5. Human Sign-off Checklist

Before you click **Approve**, you — not the AI — can say yes to all:

- [ ] I understand what this PR does and why.
- [ ] I checked every AI finding myself; I posted only real issues.
- [ ] I ran the important what-if tests myself.
- [ ] I checked security and personal-data handling (if relevant).
- [ ] I'm comfortable owning this change if it breaks in production.
