# Changelog

All notable changes to the Auditor agent.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/). Consuming repos pin a tag (e.g. `v0.1.0`) to lock behaviour.

## [0.3.0] — 2026-05-28

Sink change: findings now go to **GitHub Issues** instead of the consuming repo's inbox file. Driven by a real bug surfaced 2026-05-27: the nightly auditor's commit to `notes/inbox.md` on main caused a merge conflict that blocked an unrelated PR merge. More fundamentally, `inbox.md` is the local developer's session context — agent-generated findings belong in the triage queue (where Curator's already are), not injected into the developer's inbox.

### Why

Auditor's findings are agent-triage items with the same lifecycle as Curator's: a developer should see them in `gh issue list`, close them with `wontfix` if not worth doing, or address and close them. They are NOT session-pickup context. Filing as issues:

- Eliminates merge-conflict-on-main risk entirely (Issues API doesn't touch files).
- Unifies the triage queue across both agents — one query (`gh issue list --label cleanup`) shows everything.
- Lets `auditor:wontfix` teach the agent permanently per-file, same as `curator:wontfix`.
- Surfaces findings via PR conversation (issues link to PRs naturally) without auditor-bot pushing commits.

### Changed
- **`PROMPT.md` §5 rewritten.** "Write to the inbox file" replaced with "File GitHub issues." Adds Curator-style dedup against existing open issues with title prefix `[auditor] <category>:`, dedup window for recently-closed issues, and `auditor:wontfix` permanent skip.
- **`AGENT.md`** updated: trust level reframed (issue filing only), allowed_tools narrowed from `Bash,Read,Edit,Write,Grep,Glob` to `Bash,Read,Grep,Glob` (no file editing needed).
- **`schema.json`**: `inbox_path` removed; `dedup_window_days`, `labels.base`, `labels.per_category` added. `max_findings_per_audit` renamed to `max_findings_per_pr` for clarity.
- **`defaults.yml`**: matches schema. Default labels: `auditor`, `cleanup`, plus per-category `auditor:<category>`. Default `dedup_window_days: 30`.

### Removed
- `inbox_path` config field. The agent no longer touches any file in the consuming repo.

### Requires
- Moirai ≥ `v0.3.1` — registry default for `auditor.default_sink` flips from `inbox` to `issues`. (Adapter template unchanged; `issues` sink was already supported via Curator.)

### Migration for existing consumers

After bumping `.agents/auditor` to `v0.3.0` and `.moirai` to `v0.3.1`:

```bash
.moirai/bin/install auditor --sink issues --force
```

This regenerates the workflow with `issues: write` permission (instead of `contents: write`) and drops the inbox commit-back step. Past audit blocks in `notes/inbox.md` stay (they're already merged into history); the agent simply stops adding to them.

## [0.2.0] — 2026-05-27

Schedule-mode default. Auditor now defaults to running once nightly across all open PRs in batch, instead of firing on every PR event. The per-PR mode is still supported for hosts that prefer per-PR latency.

### Why

Per-PR auditing was a blocker in practice: the auditor-bot commit-back to PR branches created rebase friction, the on-every-push timing slowed PR cycles, and the per-PR API cost scaled with team activity instead of being bounded by the night. Coherence + tidiness aren't critical-path-blockers; they're hygiene findings that suit a nightshift cadence.

### Added
- **Two-mode support** detected from the runtime envelope: PR mode (`PR_NUMBER` set) keeps the existing single-PR behaviour; schedule mode (`PR_NUMBER` unset) enumerates open PRs via `gh pr list` and audits each.
- `max_prs_per_run` config field (default 20) — cap on PRs audited per scheduled run.
- `GH_TOKEN` documented as a required envelope var (was implicit before).

### Changed
- `AGENT.md` updated: "Two modes" section, new envelope table distinguishing PR-only vs both-mode variables.
- `PROMPT.md` rewritten with the §0 enumeration step for schedule mode. Sections 1–5 are mode-agnostic. The schedule-mode loop wraps §1–§5 per open PR; each PR's block is located/replaced/prepended independently in §5.

### Requires
- [Moirai](https://github.com/derrybirkett/moirai) ≥ `v0.3.0` — added support for the `schedule` + `inbox` trigger/sink combo (commit-back to workflow ref instead of PR head ref).

## [0.1.0] — 2026-05-27

Initial extraction from [`monospace.studio/.github/agents/auditor`](https://github.com/derrybirkett/monospace.studio/tree/main/.github/agents/auditor) into its own repo. Step 4 of the [Moirai](https://github.com/derrybirkett/moirai) walking-skeleton plan — the second agent extracted, validating that the protocol accommodates more than one.

### Added
- `AGENT.md` — persona, trust level, envelope inputs, output rules. Declares `allowed_tools: Bash,Read,Edit,Write,Grep,Glob`.
- `PROMPT.md` — scanning instructions. Reads config from `$AGENT_CONFIG_PATH`. Walks: build diff set → build neighbour set → run enabled checks → filter for signal → write idempotent block to inbox file → exit.
- `schema.json` — JSON Schema for the consuming repo's `config.yml`. Configurable: `max_findings_per_audit`, `neighbor_set_cap`, `inbox_path`, `categories`, `scan_paths`, `skip_paths`.
- `defaults.yml` — sane defaults. 8 findings max, 30-file neighbour cap, all 6 categories enabled, scans `content/ app/ components/ lib/ public/ docs/`, skips `node_modules/ .next/ pnpm-lock.yaml notes/inbox.md`.
- `README.md`, `LICENSE` (MIT).

### Changed (relative to the previous monospace.studio in-repo version)
- PROMPT reads config from `$AGENT_CONFIG_PATH` (host-injected) instead of having `notes/inbox.md` hard-coded.
- Check categories are gated by config (`categories.<name>.enabled`) rather than always all running.
- Skip paths and scan paths are configurable rather than hard-coded.

### Requires
- [Moirai](https://github.com/derrybirkett/moirai) ≥ `v0.2.0` — added support for the `pull_request` trigger and the `inbox` sink commit-back pattern.
