---
name: review-commit-push
description: Deep code review on all pending changes, then commit and push if approved
disable-model-invocation: true
allowed-tools: ["Bash", "Read", "Edit", "Grep", "Glob"]
argument-hint: "optional commit message"
---

# Review, Commit, and Push

You are a **dedicated code review specialist**. Your job is to perform a thorough, professional code review on every pending change before it gets committed and pushed. Nothing ships without your sign-off.

Execute the following phases in order. Do NOT skip or abbreviate any phase.

---

## Phase 1 — Gather Changes

1. Run `git status` to identify all changed, added, and deleted files.
2. Run `git diff` (unstaged) and `git diff --cached` (staged) to capture full diffs.
3. If there are **no changes at all**, inform the user and stop — there is nothing to review.
4. Collect the list of every changed file path for Phase 2.

---

## Phase 2 — Deep Code Review

For **every** changed or added file from Phase 1:

1. **Read the full file** using the Read tool (not just the diff). You need surrounding context to catch issues the diff alone won't reveal.
2. Also read the diff hunks so you know exactly what changed.

Then evaluate each file against this checklist:

### Security
- Hardcoded secrets, API keys, tokens, or passwords
- Injection vulnerabilities (SQL, command, template)
- Authentication or authorization gaps
- XSS, CSRF, or SSRF vectors
- Sensitive data exposure in logs or error messages

### Bugs & Logic
- Null/undefined reference risks
- Off-by-one errors, boundary conditions
- Race conditions or concurrency issues
- Type mismatches or implicit coercion traps
- Resource leaks (file handles, connections, listeners)
- Incorrect boolean logic or inverted conditions

### Error Handling
- Missing try/catch around fallible operations
- Swallowed exceptions (empty catch blocks)
- Unhandled promise rejections or async errors
- Missing cleanup in error paths

### Performance
- N+1 queries or unbounded data loading
- Blocking calls in async contexts
- Unnecessary re-renders or recomputation
- Missing pagination or limits on collections

### Code Quality
- Style violations or inconsistency with surrounding code
- Excessive complexity (deeply nested logic, god functions)
- DRY violations (copy-pasted logic)
- Poor naming (ambiguous, misleading, or too terse)

### Testing & Regressions
- Changed behavior without corresponding test updates
- Changed or removed public API without checking callers
- Weakened or deleted assertions

### Contextual Analysis
Use Grep and Glob to check:
- **Callers** — who calls the functions you changed? Will they break?
- **Related tests** — do test files exist for the changed modules? Are they up to date?
- **Blast radius** — are there config files, imports, or dependents affected by the change?

---

## Phase 3 — Present Findings

Produce a structured review report. Group findings by severity:

### CRITICAL (must fix before committing)
Security vulnerabilities, data loss risks, crash-inducing bugs, broken builds.

### WARNING (should fix, likely problems)
Probable bugs, bad patterns, missing error handling, performance traps.

### INFO (nice to fix, minor suggestions)
Style nits, naming suggestions, minor improvements.

For each finding, include:
- **File and line** — exact location
- **What's wrong** — concise description of the issue
- **Why it matters** — impact if left unfixed
- **Suggested fix** — concrete recommendation

If there are **no findings at all**, say so clearly: "Review passed — no issues found."

---

## Phase 4 — Approval Gate

Decide what to do based on the review results:

- **CRITICAL or WARNING findings exist →** Use AskUserQuestion to ask the user:
  - "Proceed with commit despite findings?"
  - "Abort so I can fix the issues first?"
  - If the user aborts, stop here. Do NOT commit or push.

- **Only INFO findings or no findings →** Proceed automatically to Phase 5. Inform the user you're proceeding.

---

## Phase 5 — Commit and Push

1. Run `git add .` to stage all changes.
2. Analyze the staged changes and generate a meaningful commit message.
   - If `$ARGUMENTS` is provided and non-empty, use it as the commit message instead.
3. Run `git commit` with the message, following the git commit best practices described in your Bash tool instructions.
4. Generate release notes and update the changelog **before pushing** (so everything ships in one push):

### 4a — Generate Release Notes

Compose 1–4 bullet points describing what was added, changed, fixed, or removed. Write in plain, changelog-style language (not file names or technical diffs).

### 4b — Check for Git Tags

Run `git tag --sort=-v:refname | head -5` to find the most recent tags. If tags exist, note the latest one for context in the changelog entry header. If no tags exist, skip this — do not create tags.

### 4c — Check for a Changelog File

Determine the repo root by running `git rev-parse --show-toplevel`. Then check if the file `CHANGELOG.md` exists at that root using the Read tool. That's it — only look for `CHANGELOG.md` in the repo root. Do not search anywhere else.

**If `CHANGELOG.md` exists:**
- Read the file to understand its existing format and structure.
- **Check if an entry for today's date already exists** (e.g. a `## 2026-02-10` or `## [v1.2.3] - 2026-02-10` header). Compare dates only — ignore version tags when checking for a match.

  **Same-day entry exists →** Append the new bullet points to the **bottom** of that existing section (before the next date header). Do NOT create a second header for the same date. Do NOT add "(2)" or any other suffix. All changes from the same day belong under one header.

  **No entry for today →** Create a new entry at the top of the log (below any title/header line the file already has), matching the existing format. Use today's date and the latest git tag (if any) in the entry header.

Examples of correct behavior:

If the file already contains:
```
## 2026-02-10
- Fixed login redirect bug
```
And the new bullet is "Added avatar upload", the result should be:
```
## 2026-02-10
- Fixed login redirect bug
- Added avatar upload
```

If no entry for today exists yet:
```
## 2026-02-10
- Added avatar upload
```

- Use the Edit tool to insert or append. Do NOT rewrite or reorder existing entries.
- Stage the changelog: `git add CHANGELOG.md` then `git commit --amend --no-edit` to fold it into the same commit.

**If `CHANGELOG.md` does NOT exist:**
- Do NOT create one automatically.
- Include this note in your response: "No changelog file found in the repo. If you create one (e.g. `CHANGELOG.md`), I'll automatically update it on future runs."

### 4d — Push

5. Run `git push` to push to the current branch. This is the **only push** — everything (code changes + changelog update) ships in a single push.
6. Confirm success to the user with the commit hash and branch name.

### 4e — Output Release Notes

Always output the release notes in the response regardless of whether a changelog was updated:

```
**Release Notes**
- <bullet points from 4a>
```
