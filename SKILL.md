---
name: openclaw-pr-workflow
description: "Guide unofficial contributor-safe OpenClaw PR work: verify behavior, make focused changes in a clean worktree, run local validation and Codex review, publish a ready PR, and handle automated feedback without using maintainer-only infrastructure."
---

# OpenClaw PR Workflow

Use this skill for ordinary contributor PR work against `openclaw/openclaw`: bug fixes, small features, tests, docs tied to code behavior, and focused follow-up on review feedback.

This is an unofficial contributor workflow. It does not override OpenClaw's repository instructions, grant maintainer permissions, or make maintainer-only infrastructure available. When this skill conflicts with `AGENTS.md`, scoped `AGENTS.md`, repository docs, or maintainer direction, follow the repository or maintainer instruction.

Do not use this workflow for release publishing, GHSA/advisory work, broad maintainer triage, landing someone else's PR, or any task that requires maintainer authority unless the user explicitly confirms that authority and asks for that workflow.

## Non-Negotiables

- Work in a separate git worktree that starts at `origin/main`.
- Never put `[codex]`, `codex:`, AI-assistance markers, or similar process labels in the PR title.
- Use a conventional-ish title and commit subject, such as `fix(scope): summary` or `test(scope): summary`.
- Create the PR as ready for review, never draft.
- Run `codex review --base origin/main` after every change batch and again before publishing.
- Wait for every `codex review --base origin/main` run to finish. Do not stop early because it is slow.
- Address every actionable local Codex review finding, then rerun `codex review --base origin/main`. Repeat until it returns no actionable feedback.
- After publishing, wait for automatic reviews to finish. Address actionable comments, push fixes, and repeat until no actionable review comments remain.
- Treat `main` freshness as a bounded checkpoint, not an infinite chase. OpenClaw `main` can move extremely fast, with thousands of commits per day; it is impossible to prove a contributor PR is always rebased on the newest remote tip. Start from `origin/main`, refresh at natural checkpoints, and rebase again only for actual conflicts, stale-base CI failures, maintainer request, or meaningful integration risk.
- Do not post maintainer-only bot-control comments from contributor PRs. This includes `@clawsweeper re-review`, `@clawsweeper review`, `@clawsweeper fix ci`, `@clawsweeper autofix`, `@clawsweeper automerge`, slash review commands, and similar bot commands. Before any public bot-control comment, verify the current GitHub actor is `OWNER`, `MEMBER`, or `COLLABORATOR`, or has `admin`, `maintain`, or `write` permission. If not, report in chat that maintainer re-review or maintainer action is needed instead; do not leave the command comment.
- Do not add PR text that implies manual verification was not done. Separate agent-run validation from user/manual verification, and do not claim the agent manually verified something it did not verify.
- Do not invoke Blacksmith/Testbox (`blacksmith testbox ...`, `pnpm testbox:*`, warmups, claims, or remote runs) as part of this skill unless the user explicitly asks for maintainer/Testbox proof. Contributor dev-loop work uses focused local validation and GitHub PR CI for broad proof.
- Do not add new config options, CLI flags, env vars, channel knobs, or user-visible switches by default. Prefer a safe default behavior fix. Add configurable surface only when maintainers explicitly ask for it, repo docs already require it, or the behavior cannot safely be defaulted.

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

## Scope For Maintainers

Before editing, run a scope preflight:

1. Identify the smallest current behavior defect or docs mismatch the PR should fix.
2. Check whether the fix can be a safe default with no new public surface.
3. If the first idea adds a config option, CLI flag, env var, channel/account knob, generated schema entry, docs option, or compatibility branch, step back and try the behavior-only fix first.
4. Add a new knob only when there is concrete evidence that two supported user groups need different behavior, or when a maintainer asked for the knob. Otherwise, the extra option increases config, docs, generated metadata, and test matrix surface for maintainers.
5. Keep the patch easy to extract. If maintainers decide to keep the behavior but remove a knob or broadening detail, treat that as accepted product direction and update the PR instead of defending the extra surface.

## Develop And Prove

- Verify source behavior before deciding on a fix: code path, tests, current shipped behavior, dependency docs/types/source, and relevant contracts.
- Keep changes focused. Add or update the smallest reliable regression coverage for bug fixes when feasible.
- Prefer hardening existing behavior behind existing contracts over adding optional behavior branches.
- Do not add or edit `CHANGELOG.md` from contributor PRs. Changelog entries are maintainer-owned; only touch them when a maintainer explicitly asks for it.
- Validation is contributor-safe by default:
  - targeted tests for the touched code
  - targeted formatter, lint, or type probes for the touched files when available
  - `pnpm changed:lanes --json` when useful to explain scope
  - `pnpm test:changed` for tests-only edits only when it stays bounded and local; stop if it fans out into broad/shared lanes
  - do not run `pnpm check:changed` as a blanket pre-handoff, pre-commit, or pre-push gate when repo instructions would route it to Testbox or when it selects broad/shared lanes; rely on GitHub PR CI for that broad matrix proof
  - `pnpm build` before push when build output, packaging, lazy/module boundaries, or published surfaces changed and the build is feasible locally
- If an `AGENTS.md` says broad gates belong in Testbox, treat that as maintainer-only infrastructure for this skill. Do not try Testbox; say that broad validation is covered by PR CI unless the user explicitly requested Testbox.
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

- Use OpenClaw's commit helper for scoped commits when it is available:

```bash
scripts/committer "fix(scope): concise summary" <file> [<file> ...]
```

- Include only intended files.
- Keep the subject concise, conventional-ish, and action-oriented.
- Add a body only when the change needs context; include root cause, validation, or compatibility notes rather than generic AI/process language.
- Do not include `[codex]` or similar markers in the commit subject.
- If the helper is unavailable, stage only intended files and use an equivalent conventional-ish `git commit` without process labels.

## Publish Ready PR

1. Refresh once against current main before pushing, unless the branch was already rebased recently and there is no evidence of conflict or stale-base risk. Do not keep rebasing only because `origin/main` advanced again while validation, review, or push work is in progress:

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
- Call out public-surface impact when relevant: `No new config surface.` or, if a knob was unavoidable, explain why a safe default was not enough.
- For human verification, state concrete manual verification details the user supplied or a neutral line such as `User manual verification: owner performs final manual verification outside this agent run.` Do not write phrases like "didn't verify a full iteration", "not manually verified", or equivalent weak-verification disclaimers.
- Do not describe missing Testbox access as a validation failure unless the user explicitly asked for Testbox. For normal contributor-style PRs, use a line such as `Broad validation: GitHub PR CI.`.

## Post-PR Review Loop

After opening or updating the PR:

1. Wait for automatic reviews and relevant checks on the exact PR head SHA. Poll at reasonable intervals.
2. Fetch review submissions, review threads, and PR comments, not just top-level comments.
3. Address every actionable automated review comment in code or with a concise evidence-backed reply. Do not trigger maintainer-only review bots as a workaround for stale or missing bot updates.
4. Resolve conversations that are fully addressed when the tool surface allows it.
5. Push fixes and rerun:

```bash
codex review --base origin/main
```

6. Wait for the updated automatic reviews again.

Repeat until automatic reviews are complete and there are no unresolved actionable comments. Stop only for a real blocker such as missing permissions, unavailable review tooling after retry, unrelated failing main CI with proof, or explicit user instruction.
