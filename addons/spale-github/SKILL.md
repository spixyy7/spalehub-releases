---
name: spale-github
description: "Universal commit-and-push methodology. Stages every local change in the current repo, writes a clean conventional commit, and pushes to the GitHub remote."
---

# SpaleGithub — Universal Commit & Push Skill

When the builder runs this skill, **commit all local changes in the current repository to GitHub**. This is a one-shot ship action — no questions, no half-measures, no leftover working tree. The end state is: clean working tree, new commit on the current branch, pushed to the remote.

This skill is **universal** — it applies to whatever repository the agent is currently inside, regardless of project, branch, or stack.

---

## Phase 1: Verify You're In a Repo

```bash
git rev-parse --is-inside-work-tree
```

- If this fails, **stop immediately** and tell the builder: "Not inside a git repository — nothing to commit."
- Do **not** run `git init` unless the builder explicitly asked you to initialize a repo.

Capture context:
```bash
git rev-parse --abbrev-ref HEAD       # current branch
git remote -v                          # confirm a GitHub remote exists
```

If there's no remote, you can still commit locally — but warn the builder before pushing fails later.

---

## Phase 2: Survey What Changed

Run these in parallel; never trust a single signal:

```bash
git status
git diff --stat
git diff --cached --stat
git log -10 --oneline                  # commit-message style for this repo
```

Read every modified, added, deleted, and untracked file's diff before staging. You write the commit message — you must understand what's in it.

**Never use `-uall`** on `git status` — it can blow up on large repos.

---

## Phase 3: Secret & Risky-File Sweep (mandatory)

Before staging *anything*, scan the surface for:

- `.env`, `.env.*` (any variant)
- `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa`, `id_ed25519`, private SSH keys
- `credentials.json`, `service-account.json`, `secrets.json`, `config.local.*`
- `.npmrc`, `.pypirc` containing tokens
- Files named `token`, `secret`, `password` (any case)
- AWS / GCP / Azure keys, Stripe `sk_live_`, OpenAI/Anthropic keys, GitHub PATs (`ghp_`, `gho_`, `ghs_`), JWT tokens, private keys

If any are present in the staged set:
1. **Refuse to stage them.** Use explicit file paths in `git add` — never `git add -A` or `git add .`.
2. Tell the builder which files were skipped and why.
3. Suggest adding them to `.gitignore`.
4. If a secret may have *already* been committed in earlier history, warn the builder loudly — that requires history rewriting and credential rotation, not just a new commit.

Also flag:
- Files >50 MB (Git LFS territory, will be rejected by GitHub).
- Build artifacts (`dist/`, `build/`, `.next/`, `node_modules/`, `target/`) — usually these belong in `.gitignore`, not commits.
- Lockfile churn that wasn't requested (`package-lock.json`, `yarn.lock`, `Cargo.lock`, `bun.lockb`).

---

## Phase 4: Decide on Commit Strategy

Default: **one commit** that bundles all related changes.

Split into multiple commits *only* if:
- Changes touch unrelated concerns (e.g., a feature + an unrelated bug fix on the side).
- The diff is large enough that a single message can't honestly describe it.

If splitting, stage and commit each logical group separately. Never amend a published commit unless the builder explicitly asked for it — always create a new commit.

---

## Phase 5: Stage Files Explicitly

Use named paths whenever possible:

```bash
git add path/to/file.ts path/to/other.ts
```

Use `git add -A` or `git add .` only when:
- You've already confirmed the secret sweep is clean.
- The set of changed files is large and *all* of it should be committed.

After staging, re-verify:
```bash
git diff --cached --stat
```

If the staged set doesn't match what you intended, unstage and redo (`git restore --staged <path>`).

---

## Phase 6: Write the Commit Message

Follow **Conventional Commits** (`type(scope): subject`) unless the repo's `git log` shows a different convention — match the repo style.

**Types**: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `style`, `build`, `ci`, `revert`.

**Subject line rules**
- ≤72 chars; ≤50 ideal.
- Imperative mood ("add", "fix", "update" — not "added", "fixes").
- No trailing period.
- Lowercase after the type prefix.
- Focus on the **why**, not just the what — well-named files already say what changed.

**Body (when warranted)**
- Wrap at 72 chars.
- Explain motivation, trade-offs, and links to issues/PRs.
- Use bullet points for multi-part changes.

**Don't**
- Don't pad with "as requested by user" boilerplate.
- Don't stuff unrelated changes into one subject.
- Don't reference the AI agent or the conversation in the message.

**Example**
```
feat(skills): add SpaleSEO methodology to built-in skills

Synthesizes 2025/2026 SEO best practices across title tags,
heading hierarchy, Core Web Vitals, structured data, and
AI-citation patterns. Wires Growth category visuals across
the skills sidebar and SpaleSkills page.
```

---

## Phase 7: Commit (HEREDOC for safety)

Always pass the message via HEREDOC so multi-line bodies render correctly:

```bash
git commit -m "$(cat <<'EOF'
feat(area): short imperative subject

Optional body explaining the why. Wrap at 72 chars.
- Bullet for individual notable change
- Another bullet
EOF
)"
```

**Never** pass `--no-verify`, `--no-gpg-sign`, or any hook-bypassing flag unless the builder explicitly asked for it. Pre-commit hooks exist for a reason — if one fails:

1. Read the hook's output.
2. Fix the underlying issue (lint, format, test failure).
3. Re-stage the fix.
4. Create a **new** commit. Never `--amend` after a hook failure — the previous commit may not have happened, and amending could destroy work.

---

## Phase 8: Push to GitHub

Determine the upstream:

```bash
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null
```

If the branch already tracks a remote, push normally:
```bash
git push
```

If it doesn't (new branch), set upstream on first push:
```bash
git push -u origin "$(git rev-parse --abbrev-ref HEAD)"
```

**Branch safety**
- If the current branch is `main` / `master` / `production` / `release/*`, **pause and confirm with the builder before pushing.** Direct pushes to protected branches are rarely intentional.
- **Never** `git push --force` or `--force-with-lease` to `main`/`master`. To any branch, only force-push when the builder explicitly asks for it, and prefer `--force-with-lease` over `--force`.

If the push is rejected (non-fast-forward):
1. `git fetch` then `git pull --rebase` to incorporate remote changes.
2. Resolve any conflicts.
3. Re-run the commit/push sequence.

---

## Phase 9: Verify the End State

```bash
git status                     # working tree should be clean
git log -1 --stat              # confirm the commit landed as expected
git rev-parse HEAD             # capture the SHA
```

Report back to the builder:
- ✅ Branch name
- ✅ Commit SHA (short)
- ✅ Files changed / lines added & removed
- ✅ Remote URL the commit was pushed to
- ✅ One-line summary of what shipped

If anything is incomplete (push pending, hook still failing, secrets skipped), say so explicitly. **Never claim "done" when the working tree is dirty.**

---

## Hard Rules

1. **Never commit secrets.** Scan before staging. When in doubt, ask.
2. **Never bypass hooks** with `--no-verify` unless the builder explicitly asked.
3. **Never amend pushed commits** unless the builder explicitly asked.
4. **Never force-push to main/master.**
5. **Never use `git add -A` or `.`** until you've completed the secret sweep.
6. **Never run destructive git commands** (`reset --hard`, `clean -fd`, `checkout -- .`, `branch -D`) without explicit builder authorization.
7. **Never modify `git config`** as part of this skill.
8. **Never commit when there's nothing to commit** — don't create empty commits.
9. **Match the repo's existing commit style** if it diverges from Conventional Commits.
10. **Stop and ask** if you can't determine intent — better to pause than to push the wrong thing.

---

## When You're Unsure

- "Is this file safe to commit?" → assume **no** until you've inspected it.
- "Should this be one commit or many?" → default to **one**, unless the diffs are clearly unrelated.
- "Should I push to this branch?" → if the branch is protected, **ask first**.
- "Did the hook actually pass?" → re-run `git status`. The commit only happened if the working tree is clean *and* the SHA changed.

---

## One-Line Distillation

> **Survey → secret-sweep → stage explicitly → commit with intent → push safely → verify clean. No shortcuts, no surprises.**
