<!--
SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
SPDX-License-Identifier: MIT
-->

# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.5] - 2026-08-05

### Added

- **`okf-verify`: when the primary source is a scan.** A PDF with no text layer is not unreachable,
  it is *unread* — and extractors return **exit 0 with an empty file**, which a script reports as
  success. Check the word count, never the status
  - **OCR output is not quotable without verification.** Recognition silently substitutes characters,
    and a wrong character *inside quotation marks* is worse than no quote — it is citable, looks
    authoritative, and nothing downstream catches it. This is the sharpest form of the corpus's
    central risk: a stale claim is visibly stale, but a quotation that is an OCR artefact reads as
    perfectly sourced
  - Record which path produced the text (`native` vs `ocr`) in the re-verification notes, so the next
    reviewer knows how far to trust the transcription

## [0.1.4] - 2026-08-05

**Graduated to the public marketplace** `jrjsmrtn-skills`, per `skills-workspace` ADR-0005's
incubator→public path and ADR-0012 **as amended 2026-08-05**.

### Changed

- Installation now via the public marketplace; README and CLAUDE.md repointed from
  `jrjsmrtn-private-skills`
- Added `LICENSE` (MIT), `SECURITY.md`, `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md`, matching the
  other public format plugins

### Notes

ADR-0012's original trigger required testing in a subject domain **unrelated to supply chain**,
naming a pattern-language bundle that does not yet exist. The amendment widens it to **subject
domain *or* bundle shape**, because a different axis delivered the test first: `ai-contribution-policies`
is **entity-keyed** where both existing corpora are concept-keyed, and three of 0.1.3's four
additions were invisible until a bundle stopped being concept-keyed. `supplychain-workspace`
ADR-0011 predicted exactly this at charter time.

**The residual risk is stated rather than resolved**: subject diversity is still one domain plus one
adjacent corpus. The tier tables and type vocabulary remain **worked examples, not defaults** — which
matters more now the plugin is public.

## [0.1.3] - 2026-08-05

Four additions, each from a failure or discovery while authoring a second OKF bundle and extending
the first — not from review of the skills in the abstract.

### Added

- **`okf-concept`: status records.** When a subject provably exists but its text is unreachable, a
  concept may document *existence, adoption, force and publication* while explicitly declining to say
  what it contains. With the three rules that keep it honest — say so at the top, never paraphrase a
  reachable superseded draft in its place, and use sources that are primary **for the claims actually
  made**. A status record makes different claims and needs *different sources, not weaker ones*
- **`okf-concept`: `stale_after` from the source's own review date.** A source stating when it will
  next be reviewed has answered the question a tier only guesses at. Record in the re-verification
  notes that the source supplied the date, or the next reviewer reads an unusual value as an error
- **`okf-concept`: "Editing concepts safely"** — the first tooling guidance in these skills for
  *changing* a bundle rather than validating one. `yq` for frontmatter, exact-match replacement with
  an assertion for bodies, `mq` for structural queries with a caution about round-trip rewriting
  - **The failure mode it names: an unasserted replace is a silent no-op.** It cannot distinguish
    "already correct" from "my pattern is wrong", and both return success. Three edits failed this way
    in one session — a folded YAML scalar where the search string spanned a line break, an indentation
    mismatch in a bullet continuation, and a link-reference style the file did not use
- **`okf-verify`: bot challenges, and what a status code stops proving.** A proof-of-work
  interstitial returns `200` with a full page and no document — and returns it for *any* path,
  including ones that do not exist. So on those hosts a status code distinguishes neither retrieval
  nor existence, and "source located, retrieval blocked" is indistinguishable from "the URL is wrong".
  Two sources were logged as blocked on exactly that reasoning; both URLs were simply wrong
- **`okf-verify`: when one source publishes two versions.** HTML and PDF, or a rendered page and a
  repository, can disagree — and then **the comparison is the verification**. Count rather than skim,
  and prove absence rather than truncation by checking for the sections that *bracket* the passage.
  Where nothing settles which governs, record the disagreement as the finding and cite both

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
