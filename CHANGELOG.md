<!--
SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
SPDX-License-Identifier: MIT
-->

# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-08-03

**Initial incubating release — 3 skills.**

The plugin exists because the same OKF conventions were living in two bundles' `CLAUDE.md` files and
being edited in lockstep. Extraction was prompted by editing both, identically, in one sitting.

### Added

- `okf-bundle` — bundle structure, the reserved-file constraints (`okf_version` is permitted only in
  the bundle-root `index.md`, and declares the *specification* revision), bundle-relative links and
  why they do not cross bundles, gate wiring including the scheduled run that expiry requires, and
  the versioning decision: **head `log.md` by release**, because OKF has no in-band content-version
  field and a git tag does not travel with a copied `knowledge/` tree.
- `okf-concept` — the spec-versus-house-convention table, the footnote↔`sources[].id` join in both
  directions, sourcing rules (a schema beats a capability page; a primary instrument beats commentary
  about it), and the `stale_after` tier table with the correction that "ratified" is a weaker
  guarantee than it sounds.
- `okf-verify` — expiry triage, re-checking against the source rather than a summary of it, the
  **version-currency sweep** as a question distinct from claim accuracy, re-tiering, a deletion
  criterion, and release stamping.

### Notes

- **Scope decision: no gate scripts.** The checkers stay in `supplychain-workspace/workspace/scripts`
  — one copy, no drift. Each skill instead ships self-contained checks needing only `python3` +
  `pyyaml`. A plugin makes gates travel for Claude Code users; it does not make a published bundle
  verifiable by a stranger who fetches it, and conflating those two would have been the wrong reason
  to move working code.
- Every shipped check was executed against a real 65-concept bundle, and the structural ones were
  negative-tested against a deliberately broken fixture — per ADR-0009, a control that has not been
  run is a claim, not a control.

[Unreleased]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/jrjsmrtn/okf-skills/releases/tag/v0.1.0
