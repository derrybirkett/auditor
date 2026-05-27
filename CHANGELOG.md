# Changelog

All notable changes to the Auditor agent.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/). Consuming repos pin a tag (e.g. `v0.1.0`) to lock behaviour.

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
