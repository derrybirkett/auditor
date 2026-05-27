# auditor

A pull-request-scoped LLM agent that surfaces coherence problems in a diff — IA drift, dead links, orphan files, copy/positioning drift, component duplication — and writes them as a dated checklist to your inbox file (default `notes/inbox.md`). The next session's `pickup`-style skill picks them up.

It observes. It does not fix.

## What this repo contains

Just the agent definition — prompt, config schema, and sane defaults. No workflow files, no config values, no host-specific wiring. Wiring lives in the consuming repo; [Moirai](https://github.com/derrybirkett/moirai) installs and orchestrates it.

```
AGENT.md         Persona, output rules, what auditor is not
PROMPT.md        Scanning instructions executed by the LLM
schema.json      JSON Schema for the consuming repo's config.yml
defaults.yml     Default values for every config field
CHANGELOG.md     Versioned changes — pin a tag in your consuming repo to lock behaviour
```

## How it runs

Auditor needs Moirai ≥ `v0.2.0` (which added support for `pull_request` triggers and the `inbox` sink commit-back pattern).

```bash
# In a consuming repo with Moirai already installed at .moirai
.moirai/bin/install auditor --trigger pull_request --sink inbox
```

That submodules this agent at its pinned tag, drops `.github/agents/auditor/config.yml` (overriding `defaults.yml`), and renders a workflow at `.github/workflows/auditor.yml` from Moirai's `claude-code-action` adapter. The workflow:

- Triggers on `pull_request` (opened, synchronize, reopened) + manual `workflow_dispatch`.
- Skips PRs from forks (can't push commits back).
- Injects the auditor envelope: `AGENT_NAME`, `AGENT_CONFIG_PATH`, `AGENT_REPO_DIR`, `TODAY`, plus the pull-request envelope: `PR_NUMBER`, `PR_BASE_SHA`, `PR_HEAD_SHA`.
- Runs the prompt with `Bash, Read, Edit, Write, Grep, Glob` (declared in `AGENT.md`).
- Commits any change to the inbox file back to the PR branch as `auditor-bot`.

## Trust level

Auditor operates at **observe** — one step beyond Curator's pure read-only stance: it can edit a single file (the configured inbox path) and the host adapter commits that edit back to the PR branch. Nothing else.

- Read-only on source code and content.
- Write-only to the inbox file.
- No edits to other files. No PR comments. No labels. No issues.

## What it checks

Six categories, scanned over the diff + a neighbour set (files referencing identifiers changed in the diff):

| Category | What it looks for |
|---|---|
| `link_integrity` | Internal links to routes that don't exist. Anchor links to missing headings. |
| `orphans` | MDX files not reachable from any route. Images referenced nowhere. Components with zero imports. |
| `ia_drift` | Routes not in nav. Nav entries pointing at dead routes. Indexes out of sync with disk. |
| `copy_drift` | Removed entities still mentioned. Conflicting positioning phrases. Stale time-bound phrases. |
| `metadata` | MDX frontmatter missing required fields. Inconsistent tags. Duplicate slugs. |
| `component_drift` | Two components doing the same job. Deprecated components in use. Inline styles where a token exists. |

Each can be toggled in the consuming repo's `config.yml`.

## License

MIT — see [LICENSE](LICENSE).
