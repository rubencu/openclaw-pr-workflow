---
name: openclaw-dev-loop
description: Implement OpenClaw bug fixes, small features, and code changes in a separate worktree from origin/main, run local validation plus repeated codex review loops, commit with repo conventions, publish a ready PR, and address automated review comments until clean.
---

# OpenClaw Dev Loop

Use this skill for ordinary OpenClaw development tasks that should end in a ready PR. Do not use it for release publishing, GHSA/advisory work, broad maintainer triage, or landing someone else's PR; use the dedicated OpenClaw skills for those workflows.

## Non-Negotiables

- Work in a separate git worktree that starts at `origin/main`.
- Never put `[codex]`, `codex:`, AI-assistance markers, or similar process labels in the PR title.
- Use a conventional-ish title and commit subject, such as `fix(scope): summary` or `test(scope): summary`.
- Create the PR as ready for review, never draft.
- Run `codex review --base origin/main` after every change batch and again before publishing.
- Wait for every `codex review --base origin/main` run to finish. Do not stop early because it is slow.
- Address every actionable local Codex review finding, then rerun `codex review --base origin/main`. Repeat until it returns no actionable feedback.
- After publishing, wait for automatic reviews to finish. Address actionable comments, push fixes, and repeat until no actionable review comments remain.
- Do not add PR text that implies manual verification was not done. Separate agent-run validation from user/manual verification, and do not claim the agent manually verified something it did not verify.

## Start From Main

1. In the source checkout, run `pnpm docs:list` if available and read only relevant docs.
2. Read `AGENTS.md` plus any scoped `AGENTS.md` for subtrees you will edit.
3. Fetch current main:

```bash
git fetch origin main
```

4. Create a fresh worktree and branch from `origin/main`:

```bash
git worktree add ../openclaw-<short-slug> -b codex/<short-slug> origin/main
```

5. Do all implementation, tests, commits, and PR work inside the new worktree. Leave unrelated dirty state in the original checkout untouched.

## Develop And Prove

- Verify source behavior before deciding on a fix: code path, tests, current shipped behavior, dependency docs/types/source, and relevant contracts.
- Keep changes focused. Add or update the smallest reliable regression coverage for bug fixes when feasible.
- Follow OpenClaw's AGENTS/docs changelog guidance for user-facing behavior, API, docs, or release-note-worthy changes.
- Prefer repo lanes:
  - targeted tests for the touched code
  - `pnpm test:changed` for tests-only edits
  - `pnpm check:changed` before handoff, commit, or push
  - `pnpm build` before push when build output, packaging, lazy/module boundaries, or published surfaces changed
- If dependencies are missing, run `pnpm install`, retry once, then report the first actionable error if still blocked.

## Codex Review Loop

After each edit batch:

1. Run relevant targeted tests or checks for the changed surface.
2. Run:

```bash
codex review --base origin/main
```

3. Wait for it to finish completely.
4. Fix every actionable finding.
5. Rerun tests/checks affected by the fix.
6. Rerun `codex review --base origin/main`.

Only exit this loop when the latest completed local Codex review has no actionable findings. If Codex review cannot run because of auth, tool, or service failure, retry once; if still blocked, report the blocker with the exact error instead of publishing as if review passed.

## Commit

- Use `scripts/committer` for scoped commits:

```bash
scripts/committer "fix(scope): concise summary" <file> [<file> ...]
```

- Include only intended files.
- Keep the subject concise, conventional-ish, and action-oriented.
- Add a body only when the change needs context; include root cause, validation, or compatibility notes rather than generic AI/process language.
- Do not include `[codex]` or similar markers in the commit subject.

## Publish Ready PR

1. Rebase on current main before pushing:

```bash
git fetch origin main
git rebase origin/main
```

2. Rerun affected checks if the rebase changes anything meaningful.
3. Run the final clean `codex review --base origin/main`.
4. Push the branch.
5. Create a ready PR, not a draft. Use `.github/pull_request_template.md` and fill it with concrete evidence.

PR title and body rules:

- Title uses the repo convention, for example `fix(scope): summary`.
- Do not include `[codex]`, `Codex`, AI-assistance markers, or similar process labels in the title.
- If repo policy asks for AI assistance disclosure, put it in the PR body, not the title.
- Describe the bug/behavior, affected surface, root cause, fix, tests, and local Codex review result.
- For human verification, state concrete manual verification details the user supplied or a neutral line such as `User manual verification: owner performs final manual verification outside this agent run.` Do not write phrases like "didn't verify a full iteration", "not manually verified", or equivalent weak-verification disclaimers.

## Post-PR Review Loop

After opening or updating the PR:

1. Wait for automatic reviews and relevant checks on the exact PR head SHA. Poll at reasonable intervals.
2. Fetch review submissions, review threads, and PR comments, not just top-level comments.
3. Address every actionable automated review comment in code or with a concise evidence-backed reply.
4. Resolve conversations that are fully addressed when the tool surface allows it.
5. Push fixes and rerun:

```bash
codex review --base origin/main
```

6. Wait for the updated automatic reviews again.

Repeat until automatic reviews are complete and there are no unresolved actionable comments. Stop only for a real blocker such as missing permissions, unavailable review tooling after retry, unrelated failing main CI with proof, or explicit user instruction.
