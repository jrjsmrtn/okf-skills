<!--
SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
SPDX-License-Identifier: MIT
-->

# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.2] - 2026-08-03

### Added

- **`okf-verify` Step 2: "When the primary source will not load".** The skill told you to open the
  document and assumed you could. Re-verifying an EU regulation showed the assumption is not safe,
  and that the failure mode is worse than a plain error: **success with no content**. `eur-lex.europa.eu`
  answers **HTTP 202 with a zero-byte body** to every non-browser client, so a fetch "succeeds",
  a script reports it fetched, and the reviewer quietly falls back to a search summary — becoming the
  thing the step exists to replace.
- Two symptoms to recognise (success-with-no-content, and the wrong representation — metadata or an
  SPA shell, large and plausible and answering nothing), the rule that an unreachable source is a
  **blocked** check rather than a downgraded one, and the guidance to record a non-obvious retrieval
  route in the project's conventions rather than in a concept: a concept carries the claim, not the
  recipe.
- Two anti-patterns: letting an unreachable source become a secondary one (the substitution is silent
  and the `verified` entry looks identical afterwards), and treating a 200 as a fetch.

### Notes

- The worked example includes a `curl` command, run verbatim before shipping. It matters that it was
  **wrong the first time**: written from memory without `Accept-Language`, it returned HTTP 400 for
  every document tried. Both headers are load-bearing, established by ablation, and verified across
  four instruments — GDPR, NIS2, the CRA and the AI Act. The skill says so, because a retrieval
  method is a control and an unrun one is a claim.

## [0.1.1] - 2026-08-03

**0.1.0 shipped guidance that violated the spec. This corrects it.**

### Fixed

- **`okf-bundle` and `okf-verify` told you to head `log.md` by release** (`## v0.5.0 — 2026-08-02`),
  calling it "the one conformant place a version can live". **OKF §9 requires date headings in ISO
  8601 `YYYY-MM-DD` form.** The claim was inferred from §5 and §8 rather than read from §9. Both
  bundles that followed it failed `okf validate` with 6 errors each.
- The conformant answer, now documented: a **release map in the `log.md` preamble**, which is
  unconstrained prose. With the sharper finding behind it — **OKF's log model is date-grouped, not
  release-grouped**, so per-entry release attribution is not expressible in a conformant log at all.
  The format has no concept of a release anywhere.

### Changed

- **Conformance is delegated to [`okfcli/okf`](https://github.com/okfcli/okf)** rather than
  reimplemented. The vendor-neutral Go CLI covers §5.1 footnote→`sources[].id`, §5.2 datetimes, §5.5
  expiry, §8 index frontmatter, §9 log headings and link resolution — more of the spec than the local
  checks did, and it found two classes of defect in a 65-concept corpus that three local gates and a
  weekly scheduled run had never surfaced.
- The skills now ship only what `okf` v0.2.1 does **not** check, measured rather than assumed: a
  footnote cited but never defined, a footnote defined but never cited, and `verified` with no
  `sources`. The coverage table is pinned to that version, with a note that upstream is active.
- `okf-concept`'s spec-versus-convention table corrected: the footnote→`sources[].id` join is a
  **spec** matter (§5.1) that `okf lint` enforces, not a house convention as 0.1.0 implied.

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

[Unreleased]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.2...HEAD
[0.1.2]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/jrjsmrtn/okf-skills/releases/tag/v0.1.0
