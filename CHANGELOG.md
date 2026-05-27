# Changelog

All notable changes to the Auditor agent.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/). Consuming repos pin a tag (e.g. `v0.1.0`) to lock behaviour.

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
