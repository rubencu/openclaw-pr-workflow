---
name: openclaw-pr-workflow
description: "Guide unofficial contributor-safe OpenClaw PR work: verify behavior, make focused changes in a clean worktree, run local validation and Codex review, prepare a PR for user review, open it as draft only after confirmation, and handle automated feedback without using maintainer-only infrastructure."
---

# OpenClaw PR Workflow

Use this skill for ordinary contributor PR work against `openclaw/openclaw`: bug fixes, small features, tests, docs tied to code behavior, and focused follow-up on review feedback.

This is an unofficial contributor workflow. It does not override OpenClaw's repository instructions, grant maintainer permissions, or make maintainer-only infrastructure available. When this skill conflicts with `AGENTS.md`, scoped `AGENTS.md`, repository docs, or maintainer direction, follow the repository or maintainer instruction.

Do not use this workflow for release publishing, GHSA/advisory work, broad maintainer triage, landing someone else's PR, or any task that requires maintainer authority unless the user explicitly confirms that authority and asks for that workflow.

## Non-Negotiables

- Work in a separate git worktree that starts at `origin/main`.
- Never put `[codex]`, `codex:`, AI-assistance markers, or similar process labels in the PR title.
- Use a conventional-ish title and commit subject, such as `fix(scope): summary` or `test(scope): summary`.
- Before committing, handing off, opening, or updating a PR, compare the commit subject/body and PR title/body with the actual diff and validation. Amend commits and edit PR metadata whenever they are stale, overbroad, or missing material behavior.
- Never leave PR-template placeholder text, instructional comments, example values, or TODO/TBD filler in the PR title or description. Preserve required headings/checklist labels only when the repo template or checker expects them; replace or remove placeholder content before showing proposed PR text, `gh pr create`, `gh pr edit`, or equivalent UI updates.
- Do not push the branch or create a PR until the user has reviewed the final branch summary, validation, and proposed PR title/body, then explicitly confirms PR creation.
- When the user confirms PR creation, open the PR as a draft. Do not mark it ready for review, run `gh pr ready`, remove draft status, or use an equivalent UI action until the user explicitly confirms ready-for-review after seeing the draft PR.
- For fork PRs, allow upstream maintainers to modify the PR branch. Do not pass `--no-maintainer-edit`; verify `maintainer_can_modify: true` after draft publication so maintainers and ClawSweeper repair/automerge lanes can push follow-up commits to the source branch once the PR is ready.
- Run `codex review --base origin/main` after every change batch and again before asking the user to confirm PR creation.
- Wait for every `codex review --base origin/main` run to finish. Do not stop early because it is slow.
- Address every actionable local Codex review finding, then rerun `codex review --base origin/main`. Repeat until it returns no actionable feedback.
- After the user confirms ready-for-review and the PR is marked ready, wait for automatic reviews to finish. Address actionable comments, push fixes, and repeat until no actionable review comments remain.
- Treat `main` freshness as a bounded checkpoint, not an infinite chase. OpenClaw `main` can move extremely fast, with thousands of commits per day; it is impossible to prove a contributor PR is always rebased on the newest remote tip. Start from `origin/main`, refresh at natural checkpoints, and rebase again only for actual conflicts, stale-base CI failures, maintainer request, or meaningful integration risk.
- Do not post maintainer-only bot-control comments from contributor PRs. This includes `@clawsweeper re-review`, `@clawsweeper review`, `@clawsweeper fix ci`, `@clawsweeper autofix`, `@clawsweeper automerge`, slash review commands, and similar bot commands. Before any public bot-control comment, verify the current GitHub actor is `OWNER`, `MEMBER`, or `COLLABORATOR`, or has `admin`, `maintain`, or `write` permission. If not, report in chat that maintainer re-review or maintainer action is needed instead; do not leave the command comment.
- Do not add PR text that implies manual verification was not done. Separate agent-run validation from user/manual verification, and do not claim the agent manually verified something it did not verify.
- Do not invoke Blacksmith/Testbox (`blacksmith testbox ...`, `pnpm testbox:*`, warmups, claims, or remote runs) as part of this skill unless the user explicitly asks for maintainer/Testbox proof. Contributor dev-loop work uses focused local validation and GitHub PR CI for broad proof.
- Do not require or preflight `gitcrawl`. `gitcrawl` is maintainer triage/discovery tooling and an optional local `gh` cache for agents; ordinary contributor PR work should proceed without it. If another OpenClaw repo instruction forces a maintainer read path for a pasted PR or issue URL, treat missing `gitcrawl` as a quiet reason to use live GitHub instead, not as a contributor setup problem.
- Do not add new config options, CLI flags, env vars, channel knobs, or user-visible switches by default. Prefer a safe default behavior fix. Add configurable surface only when maintainers explicitly ask for it, repo docs already require it, or the behavior cannot safely be defaulted.
- Treat real-behavior proof as exact-head, after-fix evidence. Historical logs can prove the old bug, but they do not prove the fix unless they come from a checkout containing the PR head.
- Treat the user's reported symptom as the proof contract for bug-fix PRs. Do not claim a PR proves or fixes that report when the live scenario exercises a different session origin, channel, client, auth path, delivery route, or user-visible workflow. Adjacent stress cases can be useful extra evidence, but they must be labeled as separate coverage and cannot substitute for the original repro without explicit user/maintainer approval.
- Treat live real-behavior proof as an explicit-request action. Do not rerun live proof just because a branch was rebased, a commit was amended or recreated, PR text changed, or checks were refreshed. If code movement or reviewer feedback makes the existing proof stale, report that fresh live proof is needed and wait for user instruction before running it.
- Do not call a PR ready from mergeability or green checks alone. Check unresolved review threads, top-level bot comments, current labels, and the exact PR head SHA.
- Do not treat automated "best possible fix" wording as proof by itself. It is only decision-grade when the review also checked the relevant owner contract, adjacent implementations, and plausible maintainer-level alternatives.

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
2. Write a one-sentence repro contract for the originally reported user-visible workflow, including the initiating surface/client, session origin, explicit route/delivery flags, auth/provider dependency when relevant, and expected visible result. Keep this sentence in the handoff notes and use it to judge proof and PR text.
3. Check whether the fix can be a safe default with no new public surface.
4. If later code review shows the patch is actually fixing an adjacent workflow instead of the repro contract, stop and re-scope before pushing, updating PR text, or calling the proof sufficient.
5. If the first idea adds a config option, CLI flag, env var, channel/account knob, generated schema entry, docs option, or compatibility branch, step back and try the behavior-only fix first.
6. Add a new knob only when there is concrete evidence that two supported user groups need different behavior, or when a maintainer asked for the knob. Otherwise, the extra option increases config, docs, generated metadata, and test matrix surface for maintainers.
7. Keep the patch easy to extract. If maintainers decide to keep the behavior but remove a knob or broadening detail, treat that as accepted product direction and update the PR instead of defending the extra surface.

## Contract And Ownership Pass

Use this pass for nontrivial runtime changes, lifecycle/events, harness behavior, provider/channel/tool contracts, dependency-backed behavior, or any PR that has sat open across meaningful `main` or dependency movement.

Before calling the patch best-available or ready:

1. Name the owning module or dependency contract for the behavior. Prefer actual exported types, schemas, source files, docs, or upstream package declarations over locally duplicated string unions.
2. Check adjacent implementations that handle the same concept, such as other harnesses, channels, providers, hooks, or dispatch paths. The question is whether the patch fits the system contract, not only whether the diff is locally correct.
3. Prefer reusing the owner contract or adding a narrow generic seam over copying event shapes, policy state, or lifecycle meanings into the touched file. If a local adapter or fallback is needed for legacy tests or synthetic fixtures, keep it explicit and covered.
4. Have the "conversation with the AI" explicitly: ask whether there is a better owner-level or contract-level fix than the narrow diff, and require file/type evidence in the answer. Do not stop at a clean patch-local review when this pass applies.
5. If the broader pass finds no better shape, say that modestly: `patch appears correct against the current owner contract`. Avoid PR text or handoff wording that says `best possible fix` unless the owner contract and alternatives were actually checked.

## Develop And Prove

- Verify source behavior before deciding on a fix: code path, tests, current shipped behavior, dependency docs/types/source, and relevant contracts.
- Keep changes focused. Add or update the smallest reliable regression coverage for bug fixes when feasible.
- Prefer hardening existing behavior behind existing contracts over adding optional behavior branches.
- Do not add or edit `CHANGELOG.md` from contributor PRs. Changelog entries are maintainer-owned; only touch them when a maintainer explicitly asks for it.
- Cover every new behavior branch introduced by the patch when feasible. If a branch cannot be covered locally, say exactly which branch remains uncovered and why; do not hide behind a green aggregate test run.
- Validation is contributor-safe by default:
  - targeted tests for the touched code
  - targeted formatter, lint, or type probes for the touched files when available
  - `pnpm changed:lanes --json` when useful to explain scope
  - `pnpm test:changed` for tests-only edits only when it stays bounded and local; stop if it fans out into broad/shared lanes
  - do not run `pnpm check:changed` as a blanket pre-handoff, pre-commit, or pre-push gate when repo instructions would route it to Testbox or when it selects broad/shared lanes; rely on GitHub PR CI for that broad matrix proof
  - `pnpm build` before push when build output, packaging, lazy/module boundaries, or published surfaces changed and the build is feasible locally
- If an `AGENTS.md` says broad gates belong in Testbox, treat that as maintainer-only infrastructure for this skill. Do not try Testbox; say that broad validation is covered by PR CI unless the user explicitly requested Testbox.
- If dependencies are missing, run `pnpm install`, retry once, then report the first actionable error if still blocked.

## Real Behavior Proof

When the user explicitly asks you to run or refresh live real-behavior proof, or tells you to satisfy a proof request from the PR template, labels, CI, or ClawSweeper:

1. Fetch/read the current proof policy from the repo before interpreting labels or schema. Start with `.github/pull_request_template.md`, `CONTRIBUTING.md`, `.github/workflows/real-behavior-proof.yml`, and `scripts/github/real-behavior-proof-policy.mjs` when present.
2. Restate the repro contract from the user's original report before designing the proof. Include the initiating client/surface, whether the session is fresh or inherited from an external channel, explicit delivery flags, provider/auth dependency, and the exact user-visible success condition.
3. Prove the failure on `origin/main` or another explicit base when the user asks for proof of the problem, or when a before/after claim would otherwise be weak. The failure proof must exercise the repro contract. If it instead uses a stronger, narrower, broader, stale-state, or adjacent scenario, label it `adjacent proof` and ask before treating it as the bug proof.
4. Prove the fix from a checkout containing the exact PR head using the same repro contract as the failure proof whenever possible. Record the head SHA, command or real workflow, environment, evidence after fix, observed result, and anything not tested.
5. If the exact repro contract is infeasible, do not weaken the claim silently. State which contract fields were not exercised, why, and whether the proof is only boundary proof or adjacent proof. Get explicit user/maintainer acceptance before using it as PR proof for the reported bug.
6. After later rebases, commit amendments, force pushes, metadata edits, or check refreshes, reuse recorded live proof only as prior after-fix evidence when the proved behavior is unchanged. Do not silently relabel old proof as exact-head proof for a new SHA. If current policy or reviewer feedback requires exact-head proof on the current SHA, say fresh live proof is needed and wait for explicit user direction before rerunning it.
7. Treat tests, mocks, snapshots, lint, typechecks, and CI as supplemental. They support real behavior proof, but they do not replace real-environment after-fix evidence when the repo requires it.
8. Match proof to the reported user path. A direct `openclaw agent` run can prove the shared gateway, harness, and tool-auth boundary, but it does not prove a channel-specific path such as TUI, Telegram, Discord, or Slack unless that channel path is exercised. If full channel proof is infeasible, say exactly which lower boundary was proven and keep `What was not tested` explicit.
9. For auth-dependent bugs, verify current auth freshness before both negative and positive proof. Use the same valid OAuth/API-key state for the base negative control and the PR-head proof whenever possible; stale-auth logs can explain a symptom but should not be used as the only bug proof.
10. For gateway/runtime proof, isolate `HOME`/`OPENCLAW_HOME`, ports, tokens, generated artifacts, and logs so stale local state or mixed proof logs cannot contaminate the result. Stop foreground gateways, TUI sessions, and other live proof processes afterward; check for leftovers when proof used long-running sessions.
11. For interactive TUI proof, run `pnpm tui` with a real PTY and verify the prompt was actually submitted. If `--message` does not visibly dispatch, type and submit the prompt in the connected TUI, then record that detail in the proof.
12. For installed external plugins or harnesses, record whether proof used the bundled implementation or the installed external package. Same model/provider labels are not enough when package resolution can change tool behavior.
13. For approval-gated external plugins such as [TweetClaw](https://github.com/Xquik-dev/tweetclaw), include the package or repository source, install command, configured placeholder variables, and whether proof stayed read-only or crossed into post/reply/DM/media/write actions. Do not include API key values, account identifiers, private message content, or live credential material in the PR body or logs.
14. Before editing the PR body, validate the proposed proof text against the repro contract and with `scripts/github/real-behavior-proof-policy.mjs` when available so the body satisfies both the user-visible claim and the parser on the first try.

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

A clean local Codex review is a patch-local signal. For changes covered by the Contract And Ownership Pass, also record the broader answer in the handoff or PR-prep notes before treating the patch as ready.

## Contributor GitHub Read Path

For ordinary contributor PR work, gather live PR facts from GitHub with `gh api`
REST endpoints instead of maintainer archive tools.

Use `gh pr diff` for the patch and a REST snapshot for structured state:

```bash
PR=<number>
REPO=openclaw/openclaw
gh api "repos/$REPO/pulls/$PR" \
  --jq '{number,title,state,draft,mergeable,mergeable_state,maintainer_can_modify,head:{ref:.head.ref,sha:.head.sha,repo:.head.repo.full_name},base:{ref:.base.ref,sha:.base.sha},user:.user.login,changed_files,additions,deletions,merged,merged_at,url:.html_url}'
gh api "repos/$REPO/pulls/$PR/files" --paginate \
  --jq '[.[] | {filename,status,additions,deletions,changes}]'
gh api "repos/$REPO/pulls/$PR/reviews" --paginate \
  --jq '[.[] | {user:.user.login,state,submitted_at,body}]'
gh api "repos/$REPO/issues/$PR/comments" --paginate \
  --jq '[.[] | {user:.user.login,created_at,body}]'
gh pr diff "$PR" --repo "$REPO"
```

Then check the exact head SHA:

```bash
HEAD_SHA=$(gh api "repos/$REPO/pulls/$PR" --jq '.head.sha')
gh api "repos/$REPO/commits/$HEAD_SHA/check-runs" \
  -H "Accept: application/vnd.github+json" \
  --jq '[.check_runs[] | {name,status,conclusion,details_url}]'
```

Use `gh pr view` only as a quick human-readable preview. Avoid depending on
`gh pr view --json` for the full workflow snapshot when it trips over GitHub
CLI GraphQL compatibility issues such as deprecated Projects Classic
`projectCards` fields; the REST endpoints above provide the same contributor
facts without requiring maintainer-only infrastructure.

## Commit

- Before committing, inspect the staged diff and review the planned commit subject/title and message against the actual change. If they no longer match the final patch, rewrite them before committing.
- Use OpenClaw's commit helper for scoped commits when it is available:

```bash
scripts/committer "fix(scope): concise summary" <file> [<file> ...]
```

- Include only intended files.
- Keep the subject concise, conventional-ish, and action-oriented.
- Add a body only when the change needs context; include root cause, validation, or compatibility notes rather than generic AI/process language.
- Do not include `[codex]` or similar markers in the commit subject.
- If later review fixes, rebases, or scope changes alter the behavior represented by the branch, amend or recreate the commit so the subject/title and message stay in sync before pushing or handing off.
- If the helper is unavailable, stage only intended files and use an equivalent conventional-ish `git commit` without process labels.

## Prepare And Gate PR

1. Refresh once against current main before pushing, unless the branch was already rebased recently and there is no evidence of conflict or stale-base risk. Do not keep rebasing only because `origin/main` advanced again while validation, review, or push work is in progress:

```bash
git fetch origin main
git rebase origin/main
```

2. Rerun affected local checks if the rebase changes anything meaningful. Do not rerun live real-behavior proof as part of the rebase unless the user explicitly asks for it.
3. Run the final clean `codex review --base origin/main`.
4. Prepare the branch summary and proposed PR title/body from `.github/pull_request_template.md`, filling it with concrete evidence.
5. Before pushing or creating any PR, hand off for user review with:
   - the final diff scope
   - validation run and results
   - local Codex review result
   - branch name and latest commit
   - proposed PR title and body
   - any remaining risks or untested branches
6. Stop and wait for explicit user confirmation to open the draft PR. A broad initial request such as "fix this and open a PR" is permission to prepare this handoff, not permission to skip the final review gate.
7. Only after the user confirms PR creation, push the branch and create a draft PR. For fork PRs, leave maintainer edits enabled. With `gh pr create`, pass `--draft` and do not pass `--no-maintainer-edit`.
8. After the draft PR exists, verify it is still draft and verify maintainer edit permission on the exact PR:

```bash
PR=<number>
REPO=openclaw/openclaw
gh api "repos/$REPO/pulls/$PR" --jq '{number,draft,head:{repo:.head.repo.full_name,ref:.head.ref},maintainer_can_modify}'
```

If the PR head is a fork and `maintainer_can_modify` is `false`, enable it before asking maintainers or ClawSweeper to repair, rebase, or automerge after the PR is ready:

```bash
gh api -X PATCH "repos/$REPO/pulls/$PR" -F maintainer_can_modify=true \
  --jq '{number,maintainer_can_modify}'
```

If GitHub refuses the update, report the exact permission error and say that maintainer/ClawSweeper push repair may need a replacement PR instead of silently continuing. If GitHub presents the broader "allow edits and access to secrets by maintainers" warning because the fork contains Actions workflows, call that out, but do not leave the setting disabled unless the user explicitly chooses that tradeoff.

After opening the draft PR, report the draft URL and stop for user review unless the user asks for more iteration. Do not mark it ready for review until the user explicitly confirms. When the user confirms ready-for-review, re-check that the latest draft PR title/body still match the final diff and validation, then use `gh pr ready` or the equivalent UI action.

PR title and body rules:

- Title uses the repo convention, for example `fix(scope): summary`.
- Do not include `[codex]`, `Codex`, AI-assistance markers, or similar process labels in the title.
- If repo policy asks for AI assistance disclosure, put it in the PR body, not the title.
- Read the final PR title and body as plain user-visible text before opening or updating the PR. Remove any template placeholder prose, HTML comments, sample text, `TODO`, `TBD`, bracketed fill-ins, or generic prompts such as `Describe the change`; do not rely on GitHub rendering or bots to hide or clean it.
- Before opening or updating the PR, compare the title and description against the final branch diff, exact-head proof, and validation actually run. Edit the title or body when the scope, behavior, proof, or tests changed during the work.
- Describe the bug/behavior, affected surface, root cause, fix, tests, and local Codex review result.
- Call out public-surface impact when relevant: `No new config surface.` or, if a knob was unavoidable, explain why a safe default was not enough.
- Fill any `Real behavior proof` section with the exact fields the current template/checker expects. Use after-fix evidence from the PR head, not only historical bug evidence.
- In `Behavior addressed`, name only the workflow actually proven by the repro contract. Do not blend the original user report with an adjacent stress case. Put extra stress/scenario coverage in a separate paragraph such as `Additional coverage`, and say explicitly when it is not required to hit the reported bug.
- Before opening, updating, or marking a PR ready, do a mismatch check: compare the original report, PR title, summary, `Behavior addressed`, proof commands, and observed result. If any one of them describes a different channel, client, session origin, delivery route, or visible success condition, stop and either rewrite the PR scope or ask whether to split/replace the PR.
- For human verification, state concrete manual verification details the user supplied or a neutral line such as `User manual verification: owner performs final manual verification outside this agent run.` Do not write phrases like "didn't verify a full iteration", "not manually verified", or equivalent weak-verification disclaimers.
- Do not describe missing Testbox access as a validation failure unless the user explicitly asked for Testbox. For normal contributor-style PRs, use a line such as `Broad validation: GitHub PR CI.`.

## Ready PR Review Loop

Use this loop only after the user confirms ready-for-review and the draft PR has been marked ready:

1. Wait for automatic reviews and relevant checks on the exact PR head SHA. Poll at reasonable intervals.
2. Fetch review submissions, review threads, top-level PR comments, labels, status rollup, and `maintainer_can_modify`. ClawSweeper or policy feedback can live in top-level comments even when GraphQL `reviewThreads` is empty.
3. Address every actionable automated review comment in code or with a concise evidence-backed reply. If the current diff already resolves a stale comment, say that with evidence instead of changing code unnecessarily. Do not trigger maintainer-only review bots as a workaround for stale or missing bot updates.
4. Resolve conversations that are fully addressed when the tool surface allows it.
5. Push fixes and rerun:

```bash
codex review --base origin/main
```

6. Wait for the updated automatic reviews again.

Repeat until automatic reviews are complete and there are no unresolved actionable comments. Stop only for a real blocker such as missing permissions, unavailable review tooling after retry, unrelated failing main CI with proof, or explicit user instruction.

## Existing PR Maintenance

For existing PR refreshes, sweeps, and conflict triage:

- Locate the active worktree with `git worktree list --porcelain` and `git branch -vv` before editing; do not use the root checkout out of convenience if the PR already has a worktree.
- For fork PRs, check `maintainer_can_modify` before any maintainer handoff, rebase request, repair lane, or automerge request. If it is `false`, set it with the REST patch above or report the blocker clearly; do not ask for ClawSweeper repair/automerge on a branch it cannot push to.
- Use `git merge-tree --write-tree --name-only origin/main <pr-ref>` or the closest current equivalent to separate real code conflicts from `CHANGELOG.md`-only drift.
- Treat `CHANGELOG.md`-only conflicts or stale changelog comments as maintainer-owned drift unless a maintainer explicitly asks the contributor to change the changelog.
- Rebase only for real conflicts, stale-base check failures, maintainer request, or a concrete validation reason.
- For long-lived PRs, do not just resolve conflicts and rerun tests. Re-check whether `main` or dependency movement changed the owner contract, adjacent harness behavior, or the right place for the fix.
- After any maintenance change, re-check the current commit subject/title, commit message, PR title, and PR description against the actual branch diff and latest validation. Amend/edit stale text before asking the user to create a draft PR or mark an existing draft PR ready.
- For repo-wide sweeps, finish with a fresh full inventory of open PRs, mergeability, status rollup, unresolved review threads, top-level bot comments, and labels before handing off.
