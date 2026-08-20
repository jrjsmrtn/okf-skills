<!--
SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
SPDX-License-Identifier: MIT
-->

# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.11] - 2026-08-20

**Four faults from a corpus-wide verification pass that these skills did not already cover** — and
the notable thing is how few there were. The pass rebuilt a quotation checker from scratch and hit a
bug `okf-verify` had already documented, because the plugin was not surfacing. **Most of what that
pass "discovered" was already here.**

### Added

- **`okf-verify`: mistrims run in one direction, and that is the diagnostic.** Across one corpus,
  every mistrimmed quotation ended where the surrounding *argument* wanted the sentence to end. It is
  not transcription noise but the argument leaking into the evidence — which is why re-reading never
  catches it and the writer is the last person able to.
- **`okf-verify`: do not quote a figure from a page that recomputes.** One statistics page
  recalculated every payday *and* rendered amounts in the viewer's currency; the same payday read
  twice gave two different numbers. Report with a read-on date instead.
- **`okf-concept`: three anti-patterns** — a number in prose that a gate already counts; a
  verification note that outlives its evidence; and a comparison table captioned only at the foot of
  the page, when tables are the most quotable and least sourced thing in a concept.
- **`okf-bundle`: if the sources cannot be followed, the index must say so.** A bundle whose
  citations point at reserved documentation hosts is unverifiable by construction. That is a
  legitimate trade for anonymised material and it is invisible from inside the bundle — a reader sees
  citations throughout and never learns that following one is impossible.

### Notes

- Deliberately **not** added: guidance to check quotations mechanically, to draft from the downloaded
  source rather than a summary, and to avoid quoting your own claim in a re-verification note. All
  three are already here, with better evidence than the pass produced. Restating them would lengthen
  the skills without making them more likely to be read.


## [0.1.10] - 2026-08-18

**A record of what you could not retrieve ages like any other claim, and nothing in this skill said
so.** The previous release taught the cycle to distrust a source that loads and does not support the
claim. This one teaches it to distrust its own blocked list.

Found by re-testing three standards a bundle had carried as *in scope and unsourced*. All three
retrieved on the first attempt — one with a bare `curl` against the exact URL recorded as blocked.
The classification had become a fact because it was written down, and writing it down was the last
time anyone had looked.

### Added — all in `okf-verify`, under *When the primary source will not load*

- **A blocked finding is a measurement with a date, and it decays.** Re-test the blocked list every
  cycle. Record a block as an observation with its date and method (*"returned a challenge page on
  2026-08-14 via curl"*) rather than as a property of the document (*"is served only to browsers"*),
  because the second reads as settled. And when a re-test succeeds, **say you cannot tell why** —
  whether the finding was wrong or the world changed is usually unrecoverable, and picking the
  flattering answer is how a corpus loses the ability to correct itself.
- **When the standards-body edition is a stub, look for the original submission.** A specification
  that became a standard usually has an earlier, freely readable form. One document was recorded
  unsourceable on the strength of a seven-page table of contents served under the body's title, while
  that same body mirrored the full original submission under a different path.
- **Four kinds of unavailability, two of which are not the verifier's to resolve** — sold,
  bot-blocked, registration-walled, encumbered. The last two turn on terms that are frequently
  invisible until you have agreed to something, so **do not register an account, accept terms, or
  agree to a licence on the bundle owner's behalf.** They are recorded as *blocked pending a human
  decision*, a different status from *unsourced*: the missing ingredient is not effort.

### Not added, deliberately

Two candidate lessons from the same pass were already covered, and are not restated: checking
quotations against **the union** of a concept's sources rather than one file, and recognising a
landing page or an SPA shell as *the wrong representation*. Both were rediscovered the hard way,
which is evidence the existing text is right and went unread — not a reason to write it twice.

## [0.1.9] - 2026-08-15

**A quotation is the only part of a concept that cannot be judged by reading it.** A stale claim
looks stale; a misquotation looks perfect, in every reading, forever.

### Added

- **`okf-verify`: check every quotation mechanically, and keep the marks for a source.** The skill
  already covered OCR artefacts as the sharpest case; this is the general one, with the check itself
  and — more useful — **the normalisation that must happen before the check can judge anything**:
  - RFC page furniture, hyphenation across a line break, blockquote markers and emphasis inside the
    quoted span all produce **false negatives**. Every one was observed
  - **Rendered versus raw link text is a real fault**, not a false negative: a quotation crossing a
    markdown link silently drops markup the source contains. The section says which is which
- **The trap specific to re-verification**: a note about a claim you just revised invites quoting
  your own wording, which in a document where quote marks mean *the source wrote this* silently
  promotes it to sourced. Observed three times in one day, every one caught by the mechanical check
  and **none by reading the file**
- **The process finding that outweighs the check**: draft from the downloaded source, not from a
  summary. A tranche written from fetch summaries needed four quotation repairs; the next, written
  from local copies, needed one — a straight apostrophe where the source had a typographic one

### Notes

The snippet in the section was corrected **after being run**: it checked a single source file, so a
concept citing three reported two-thirds of its quotations as faults. That is the same class of error
as the normalisation table one level up — the check was wrong, not the text — and it is recorded in
the section rather than quietly fixed.

## [0.1.8] - 2026-08-14

**One rule, from four instances of the same failure in a single week.** Every example below is
something that happened, not a hazard imagined for the guidance.

### Added

- **`okf-bundle`: an index that summarises the corpus is a second copy of it.** The skill already
  said no gate reads an `index.md` heading; this is the same gap deeper. **No gate compares an
  index's claims to the corpus it indexes.** An index that only links onward cannot drift; one that
  characterises the collection — how many concepts, how many verified, what state the whole is in —
  holds facts that live in the concept files
  - **A count outlived the corpus**: an index read *"seven of the eight concepts are verified"* when
    there were nine and eight
  - **A count corrected to a named exception, falsified by the same commit.** *"Every concept but one
    is verified"* is the right instinct — an exception does not go stale the way a count does — and it
    was wrong on arrival, because the commit that wrote it added a second unverified file. **A named
    exception is robust against later change and not against the change introducing it**
  - **A summary survived two tagged releases after ceasing to be true**, because a verification pass
    edits concepts and the index is not what you are editing
  - The guidance: **describe the kind of evidence rather than quantify it.** A distribution stays
    true when a concept is added; a tally does not. If a number is genuinely the point, put it where
    it is derived — a command the reader can run — not in a sentence someone must maintain
- **`okf-verify`: re-read `index.md` and any `landscape.md` before tagging**, added as a step in the
  release stamp. A verification pass is precisely when a summary goes stale, and both a stale index
  and a `landscape.md` still calling a concept *"the concept that failed its verification"* were
  observed at exactly that point

### Notes

The two skills cross-reference rather than repeat: `okf-verify` states the step and points at
`okf-bundle` for why.

**Deliberately excluded**, as in 0.1.7: the checker names and hook wiring that enforce this
elsewhere. This plugin ships no gate scripts, and naming them would imply tooling it does not carry.

## [0.1.7] - 2026-08-14

Two gaps found by auditing three bundles built in one week against what these skills already said.
Both were absent rather than wrong.

### Added

- **`okf-bundle` now covers the type vocabulary**, which the skill had not mentioned at all. OKF
  requires `type` and **constrains no value**: `type: Bananas` passes `okf validate` and `okf lint`
  with zero errors, zero warnings and zero findings — run against a throwaway bundle rather than
  taken from a note, and confirmed live by removing the field, which fails. So the vocabulary is a
  convention and nothing enforces it by default
  - The section is about checking it in **both directions**. Used-but-undeclared catches a typo;
    declared-but-unused catches drift, and the second is the one usually skipped
  - It covers the case a one-directional check cannot see: **a type stranded when its only concept
    moves to another bundle**. The same check fires on the receiving bundle, which must declare the
    arriving type first
  - And the rule that follows: **do not declare a type before a concept uses it**, because a new type
    should be a decision rather than a side effect
- **`okf-verify` now names the outcome between verified and unchecked** — a source that loads, is the
  right document, and does not support the claim. The existing section covers a source you cannot
  read; this one is more dangerous because everything succeeds
  - Absent `verified` conflates **never checked** with **checked and unsupported**, and only the
    second tells the next person not to repeat the lookup
  - Three routes out, in order: a different *kind* of source, a demonstration reproducible against a
    named tool and version, or demoting the claim in place and leaving it visible
  - Carries the heuristic that produced it: **project documentation describes purpose, not failure
    modes.** A tool's own page states what a mechanism is for and is nearly silent on what goes wrong
    with it — including the failure most discussed in practice. A tool's documentation is evidence
    about its documentation
  - States that **removing a citation is a legitimate outcome**. A source retained because it looks
    supporting is worse than none, being the one a reader checks least

### Notes

**Three of today's errors were already documented here and happened anyway** — citing a URL that
404s, citing a landing page that supports nothing, and putting emphasis inside a quotation. All three
are in `okf-verify` already. The content was not the gap; reaching for it was. Worth knowing before
assuming that more guidance prevents more mistakes.

**Deliberately excluded**: the checker names and hook wiring used to enforce these rules. This plugin
ships no gate scripts, and naming them would imply tooling it does not carry.

## [0.1.6] - 2026-08-05

### Fixed

- **`okf-bundle` now states that no gate reads an `index.md` heading**, and that the link checker does
  not open `index.md` at all — so a broken link *inside* an index is invisible to it too. These are
  the files where the tooling stops helping, and the section previously listed their constraints
  without saying that nothing enforces them
  - Observed in a bundle that shipped four of six category indexes headed `# Uarchitecture`,
    `# Uinception`, `# Ulearning`, `# Uorchestration`. Every gate passed; only the two written by hand
    were correct
  - **The cause is not an OKF problem, which is why the skill now names it.** The headings were
    generated with `sed 's/^./\U&/'`; **`\U` is a GNU sed extension** and BSD sed — the default on
    macOS — emits the letter literally. It fails plausibly rather than loudly, which is how it survived
    review

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

[Unreleased]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.6...HEAD
[0.1.6]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.5...v0.1.6
[0.1.5]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.4...v0.1.5
[0.1.4]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.3...v0.1.4
[0.1.3]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.2...v0.1.3
[0.1.2]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/jrjsmrtn/okf-skills/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/jrjsmrtn/okf-skills/releases/tag/v0.1.0
