# OpenClaw PR Workflow

An unofficial Codex skill for producing clean, contributor-safe pull requests against
[`openclaw/openclaw`](https://github.com/openclaw/openclaw).

It guides an agent through a practical PR flow:

- start from a clean worktree based on `origin/main`
- verify the bug or behavior before changing code
- keep the diff focused
- prefer safe default behavior over new config or option surface
- produce real behavior proof from the exact PR head when explicitly asked, without rerunning live proof just because a branch was rebased or a commit changed
- run targeted local validation and `codex review --base origin/main`
- keep commit titles/messages and PR titles/descriptions synced with the actual diff and validation
- prepare the final branch and proposed PR text for user review before any PR is created
- open the PR as draft only after confirmation, and mark it ready only after a second confirmation
- keep fork PRs editable by upstream maintainers so maintainer and ClawSweeper repair lanes can push follow-up commits
- address threaded and top-level automated review feedback without using maintainer-only infrastructure
- classify changelog-only drift separately from real code conflicts during PR maintenance
- record install source, package identity, and approval boundaries when proof uses an external OpenClaw plugin such as [TweetClaw](https://github.com/Xquik-dev/tweetclaw)

It does not require `gitcrawl`. That tool is useful for maintainer
triage/discovery and optional local `gh` caching, but contributor PR work should
still run cleanly with live `gh`/`gh api`. For live PR state, the skill prefers
contributor-safe `gh api` REST reads and `gh pr diff`; `gh pr view` is only a
quick preview path because older GitHub CLI GraphQL queries can hit deprecated
Projects Classic fields such as `projectCards`.

This repository is personal workflow guidance. It is not official OpenClaw policy,
does not grant maintainer permissions, and does not replace the repository's
`AGENTS.md`, scoped instructions, docs, or maintainer direction.

## Install

Clone this repository as a Codex skill:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/rubencu/openclaw-pr-workflow.git ~/.codex/skills/openclaw-pr-workflow
```

For OpenClaw workspace-local skill discovery, also clone or copy it under:

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/rubencu/openclaw-pr-workflow.git ~/.agents/skills/openclaw-pr-workflow
```

## Use

Ask Codex to use the OpenClaw PR workflow for ordinary contributor PR work:

```text
Use the openclaw-pr-workflow skill to fix this OpenClaw bug and prepare it for my review.
```

Use a different workflow for releases, security advisories, broad maintainer triage,
or landing someone else's PR.
