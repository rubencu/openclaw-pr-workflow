# OpenClaw PR Workflow

An unofficial Codex skill for producing clean, contributor-safe pull requests against
[`openclaw/openclaw`](https://github.com/openclaw/openclaw).

It guides an agent through a practical PR flow:

- start from a clean worktree based on `origin/main`
- verify the bug or behavior before changing code
- keep the diff focused
- run targeted local validation and `codex review --base origin/main`
- publish a ready PR with concrete evidence
- address automated review feedback without using maintainer-only infrastructure

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
Use the openclaw-pr-workflow skill to fix this OpenClaw bug and open a ready PR.
```

Use a different workflow for releases, security advisories, broad maintainer triage,
or landing someone else's PR.

