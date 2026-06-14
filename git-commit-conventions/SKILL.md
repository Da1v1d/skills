---
name: git-commit-conventions
description: Git commit message format and safe commit workflow. Use when creating, amending, or drafting Git commits and Conventional Commit messages.
---

# Git commit conventions

Use this rule when the user asks to commit, amend, or write commit messages.

## When to commit

- **Only create commits when the user explicitly asks.** If unclear, ask first.
- **Always prefer one line commits** If big feature was implemented, then multiple
- **Always confirm before committing.** After staging and drafting the message, show the user what will be committed and the proposed message, then **wait for explicit approval** (e.g. “yes”, “go ahead”, “commit”) before running `git commit`. Do not commit in the same turn as the proposal unless the user already approved that exact message and file set.
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
2. Stage only relevant files; draft message from **all** staged changes.
3. **Confirm with the user** — post a short summary:
   - Proposed commit message (subject + body if any)
   - Files to be committed (paths from `git status` / staged diff)
   - Ask: “Proceed with this commit?” and **stop** until they approve.
4. Only after approval, commit with a HEREDOC (no `-i` flags):

```bash
git commit -m "$(cat <<'EOF'
feat(scope): short summary

Optional body explaining why.

EOF
)"
```

5. Run `git status` after commit to confirm success.

## Safety (do not skip)

- Never add and run `git commit` or `git add` by yourself, it's only my responsibility
- Never update git config; never `--no-verify` unless the user asks.
- **Never push.** The user pushes to remotes manually. Do not run `git push`, `git push -u`, or publish branches unless the user explicitly asks in that conversation.
- Never force-push to `main`/`master` (or any branch) without explicit request and a clear warning about rewriting remote history.
- **Amend** only when: user asked for amend, OR hook auto-modified files after a successful commit you made, HEAD is yours and unpushed.
- If commit **failed** or was **rejected by a hook**, fix and make a **new** commit — do not amend.

## Pre-commit hook failures

Fix the reported issue, re-stage if needed, then commit again with a **new** commit (not amend unless amend rules above apply).
