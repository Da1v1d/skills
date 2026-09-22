---
name: git-commit-conventions
description: Git commit message format and safe commit workflow. Use when creating, amending, or drafting Git commits and Conventional Commit messages.
---

# Git commit conventions

Use this rule when the user asks to commit, amend, or write commit messages.

## When to commit

- Prefer one-line commit messages. Add a body only when the _why_ is not obvious from the subject — and lead with the one-liner when presenting both.
- **One commit = one logical change.** If the subject needs areas joined by "and", or the body enumerates changes that don't share a reason, propose separate commits and show each message.
- Draft commit messages and commit commands only. The user is responsible for staging and running commits.
- Do not commit secrets (`.env`, credentials, keys). Warn if the user tries to include them.
- Do not create empty commits when there are no changes.

## Commit message format

Never add `Co-authored-by` trailers or other authorship credits for Claude, Codex, ChatGPT, or any AI assistant to any commit message, including drafts and amend messages. This restriction applies even when repository history or templates include AI attribution.

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

**Summary:** lowercase, imperative, no trailing period. Target ≤50 chars, hard cap 72.

**Body:** **why** / user-visible outcome, never a file list. **Hard-wrap at 72 columns — `-m` does not wrap for you.** Any body longer than one short sentence must be drafted with a heredoc, not `-m`:

```
git commit -F - <<'EOF'
docs(skills): tighten api and ui folder rules

Rules had drifted from what the codebase does; each entry now names the
behavior it forbids.
EOF
```

`-m "…" -m "…"` is acceptable only when each `-m` is itself under 72 chars (each becomes its own paragraph, still unwrapped).

**Examples:**

```
feat(invoice): add line-item quantity stepper
fix(auth): prevent crash when token is missing
ui(profile): refine account header spacing
refactor(shared): extract date formatting helper
chore: bump expo sdk
```

Match recent repo style when present (`git log -10 --oneline`); repo style wins over the length target when the two conflict.

## Workflow before committing

1. Run in parallel: `git status`, `git diff` (staged + unstaged), `git log -10 --oneline`.
2. Identify staged files. If nothing is staged, identify relevant unstaged files and draft from the intended change set without staging anything.
3. Post one draft block, in this order:
   - **Message** — the one-liner first; show a body variant only if recommending one.
   - **Files** — relevant or already staged, listed exactly once. If they also appear in the command below, don't list them above it too.
   - **Flags** — secrets, generated files, unrelated changes to exclude, or a proposed split.
   - **Command** — `git add …` and `git commit …` on separate lines, never chained with `&&`, so the user can inspect the index before committing.

## Safety (do not skip)

- Never run `git add` or `git commit`; staging and committing are only the user's responsibility.
- Never update git config; never `--no-verify` unless the user asks.
- **Never push.** The user pushes to remotes manually. Do not run `git push`, `git push -u`, or publish branches unless the user explicitly asks in that conversation.
- Never force-push to `main`/`master` (or any branch) without explicit request and a clear warning about rewriting remote history.
- Draft amend messages only when the user asks for amend.
- If a commit **failed** or was **rejected by a hook**, fix the issue, say what needs re-staging, and draft a **new** commit message — do not suggest amend unless the user explicitly asks.
