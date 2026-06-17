---
name: git-commit-conventions
description: Git commit message format and safe commit workflow. Use when creating, amending, or drafting Git commits and Conventional Commit messages.
model: Haiku
---

# Git commit conventions

Use this rule when the user asks to commit, amend, or write commit messages.

## When to commit

- Prefer one-line commit messages. Use a body only for large or non-obvious changes.
- Draft commit messages and commit commands only. The user is responsible for staging and running commits.
- Do not commit secrets (`.env`, credentials, keys). Warn if the user tries to include them.
- Do not create empty commits when there are no changes.

## Commit message format

Use **Conventional Commits** in the imperative mood:

```
<type>(<optional scope>): <short summary>

<optional body — why, not a file list>
```

**Types:**

| Type       | Use when                                                      |
| ---------- | ------------------------------------------------------------- |
| `feat`     | New user-facing capability or behavior                        |
| `fix`      | Bug fix (incorrect behavior, crash, regression)               |
| `refactor` | Internal restructure with no intended behavior change         |
| `docs`     | Documentation only (README, comments, Cursor rules text)      |
| `ui`       | Visual interface changes — layout, colors, spacing, styling   |
| `style`    | Formatting only — whitespace, lint autofix, no logic change   |
| `test`     | Adding or updating tests                                      |
| `chore`    | Routine maintenance — deps, scripts, tooling, repo hygiene    |
| `build`    | Build tooling or bundler config (Expo, Metro, tsconfig build) |
| `ci`       | CI/CD pipelines (GitHub Actions, EAS, hooks in CI)            |

**Summary:** lowercase, no trailing period, ~50 chars; focus on **why** / user-visible outcome.

**Examples:**

```
feat(invoice): add line-item quantity stepper
fix(auth): prevent crash when token is missing
ui(profile): refine account header spacing
refactor(shared): extract date formatting helper
chore: bump expo sdk
```

Match recent repo style when present (`git log -10 --oneline`).

## Workflow before committing

1. Run in parallel: `git status`, `git diff` (staged + unstaged), `git log -10 --oneline`.
2. Identify staged files. If nothing is staged, identify relevant unstaged files and draft from the intended change set without staging anything.
3. Post a short summary:
   - Proposed commit message (subject + body if any)
   - Files that appear relevant or are already staged
   - Any secrets, generated files, or unrelated changes the user should exclude
4. If the user asks for a command, provide a copyable command for the user to run. Do not run it yourself.

```bash
git commit -m "$(cat <<'EOF'
feat(scope): short summary

Optional body explaining why.

EOF
)"
```

5. If the user reports a hook failure, help fix the issue and draft a new commit message or command.

## Safety (do not skip)

- Never run `git add` or `git commit`; staging and committing are only the user's responsibility.
- Never update git config; never `--no-verify` unless the user asks.
- **Never push.** The user pushes to remotes manually. Do not run `git push`, `git push -u`, or publish branches unless the user explicitly asks in that conversation.
- Never force-push to `main`/`master` (or any branch) without explicit request and a clear warning about rewriting remote history.
- Draft amend messages only when the user asks for amend.
- If commit **failed** or was **rejected by a hook**, fix and draft a **new** commit message — do not suggest amend unless the user explicitly asks.

## Pre-commit hook failures

Fix the reported issue, tell the user what to re-stage if needed, then draft a **new** commit message or command. Do not suggest amend unless the user explicitly asks.
