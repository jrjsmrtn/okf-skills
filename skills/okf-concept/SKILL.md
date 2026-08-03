---
# SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
# SPDX-License-Identifier: MIT
name: okf-concept
description: >
  Author or revise a single concept in an Open Knowledge Format bundle.
  Use when adding a concept to an OKF bundle, correcting one, deciding whether
  a subject earns a file at all, choosing its `stale_after`, or wiring per-claim
  attribution between footnotes and `sources`.
metadata:
  version: "0.1.0"
license: MIT
---

# Author an OKF Concept

One concept is one Markdown file with YAML frontmatter. This skill covers writing a new one or
revising an existing one so that **every claim in it can be traced to the source that supports it**,
and so that a reader can tell how long the claim is expected to hold.

## When to Use

- Adding a concept to an existing OKF bundle
- Correcting a concept whose claim turned out to be wrong or superseded
- Deciding whether a subject earns a concept file at all
- Choosing or revising a `stale_after`
- Diagnosing why the bundle's attribution gate is failing on a file

**Not for**: re-verifying a corpus on a cycle (`okf-verify`), or standing up a new bundle
(`okf-bundle`).

## Required Inputs

1. **The bundle root** — the directory containing `index.md`
2. **The subject** — what the concept is about
3. **At least one primary source** for every claim the concept will make

## Spec versus convention

Keep these apart. Confusing them is how a house style gets defended as conformance, and how a real
conformance break gets waved through as taste.

| | Required by OKF v0.2 | House convention |
|---|---|---|
| Parseable YAML frontmatter | **yes** | |
| Non-empty `type` | **yes** | |
| Footnote label is the join key into `sources[].id` | **yes** — §5.1 | |
| `title`, `description`, `resource`, `tags`, `status` | no — optional families | used throughout |
| `verified`, `stale_after`, `generated` | no — optional families | **load-bearing here** |
| Every footnote *definition* is referenced | no | ours |
| A `stale_after` tier table | no | ours |

Everything in the right-hand column is a choice a bundle makes. Write it down in the bundle's own
`CLAUDE.md` so the next author inherits the choice rather than re-deriving it.

## Workflow

### Step 1 — Decide whether it earns a file

**Only decompose subjects that carry content.** A bundle of one-line stubs is worse than one page of
prose: it multiplies files to maintain while adding nothing a reader could not have read in place.

If a subject has nothing but a definition, leave it out until sourcing earns it a file. If it has a
definition plus a distinction that changes what a reader would do, it earns one.

### Step 2 — Choose the type

`type` is the one required key. A bundle's type vocabulary is its own; keep it small and keep each
value meaning one thing. The vocabulary in use in the reference corpus:

`BOM Type` · `Format` · `Identifier` · `Specification` · `Data Source` · `Tool` · `Practice` ·
`Organization` · `Explanation` · `Attack` · `Regulation`

Note `Specification` and `Regulation` are deliberately distinct: a specification *describes*, a
regulation *obliges*. Collapsing them loses the distinction that makes the second worth recording.

### Step 3 — Source every claim, then write

Order matters. Writing first and sourcing afterwards produces fluent prose that the sources turn out
not to support — and the rewrite costs more than the research would have.

Two sourcing rules, both learned by getting them wrong:

- **A capability page is a weaker source than a schema.** A vendor's feature page lists what
  marketing found worth listing. In the reference corpus, CycloneDX's CBOM page named three
  cryptographic asset types; `bom-1.6.schema.json` has four, and the missing one changed what the
  concept claimed. Prefer the schema, the RFC, the enacting text, the statute.
- **Cite the primary instrument, not commentary about it.** A webinar deck describing a superseded
  draft is not a source for what the current rule requires.

### Step 4 — Wire attribution

The footnote label **is** the join key. A footnote whose label is not a `sources[].id` attributes
nothing while looking like it does — the reader sees a citation, the consumer resolving attribution
programmatically finds nothing.

```markdown
---
type: Specification
sources:
  - id: slsa-spec
    title: 'SLSA v1.2 specification'
    resource: https://slsa.dev/spec/v1.2/
---

The Source track grades a revision, not the artifact a consumer fetched.[^slsa-spec]

[^slsa-spec]: [SLSA v1.2 specification](https://slsa.dev/spec/v1.2/)
```

Three failure modes, all invisible on rendered output:

| Fault | What the reader sees |
|---|---|
| Label is not a `sources[].id` | a citation that resolves to nothing |
| Label referenced, never defined | a dangling marker |
| Label defined, never referenced | **nothing at all** — the source silently vanishes from the page |

The third is the quiet one. Run the check in Validation rather than reading for it.

**An uncited `sources[].id` is legitimate** and should not be flagged: a narrative concept carries a
bibliography rather than per-claim attribution. Only the footnote→source direction is an error.

### Step 5 — Choose `stale_after` by volatility, not importance

Pick the tier that matches how fast **the claims in this concept** move. A default applied without
thinking is how one corpus ended up with 40 of 61 concepts sharing a single date — a number reached
for, not a judgement made.

| Tier | For |
|---|---|
| ~3 months | draft or beta specifications under active revision |
| ~4 months | version and capability claims about actively-shipping software; coverage counts that grow |
| ~6 months | registries and rosters; enums that move with a spec version |
| ~12 months | definitions, ratified specifications, structural mechanisms, attack mechanics |
| ~24 months | durable rationale — arguments rather than facts |

**"Ratified" is weaker than it sounds.** Two concepts in the reference corpus were filed at 12 months
on the reasoning that ratified specifications do not move; the specification then shipped two minor
versions and rewrote its threat taxonomy, reassigning letters that other concepts cited by letter. A
specification under active development belongs at ~6 months whatever its status field says.

Clustering *within* a tier is fine and intended — concepts that share a volatility class tend to
share sources, so re-verifying them in one sitting is cheaper. Clustering *across* classes, because
nobody chose, is the failure this replaces.

### Step 6 — Stamp `verified` only if you verified

`verified` records **an act**, not a formatting step. Add an entry only when the claims were checked
against the source in this sitting. Absent is the honest default, and a corpus full of migrated prose
nobody has re-checked should say so.

Moving a `stale_after` because the volatility class was misjudged is **not** verification and earns
no `verified` entry. See `okf-verify`.

## Validation

Run against the file you just wrote. Requires `pyyaml`.

```bash
python3 - path/to/concept.md <<'PY'
import re, sys, yaml
src = open(sys.argv[1]).read()
_, fm, body = src.split('---', 2)
meta = yaml.safe_load(fm) or {}
if not meta.get('type'):
    print("FAIL: `type` is missing or empty — the one key OKF requires"); sys.exit(1)
ids  = {s['id'] for s in (meta.get('sources') or [])}
refs = set(re.findall(r'\[\^([^\]]+)\](?!:)', body))
defs = set(re.findall(r'^\[\^([^\]]+)\]:', body, re.M))
fail = False
for label, s in [("cited but not a sources[].id", refs - ids),
                 ("cited but never defined",      refs - defs),
                 ("defined but never cited",      defs - refs)]:
    if s:
        print(f"FAIL {label}: {', '.join(sorted(s))}")
        fail = True
if not fail:
    print(f"attribution OK ({len(refs)} footnotes, {len(ids)} sources)")
sys.exit(1 if fail else 0)
PY
```

Expected on a clean concept: `attribution OK (N footnotes, M sources)`, exit 0. Any `FAIL` line
exits 1, so this works unmodified as a hook or CI step.

If the bundle ships a full gate suite, run that too — this check covers one file, the suite covers
the corpus and its links.

## Anti-Patterns

- **Writing first, sourcing after.** Produces prose the sources do not support.
- **Bumping `stale_after` without re-checking.** No gate can tell the difference, which is exactly
  why it has to be a rule.
- **Stamping `verified` for a whole tier because a review "covered" it.** Then the field means
  "someone looked at the tier", not "someone checked this concept".
- **A `stale_after` chosen by importance.** Important and fast-moving are unrelated axes.
- **Bundle-relative links across bundles.** A leading `/` means *this* bundle's root and does not
  cross a bundle boundary, even when two bundles sit side by side on disk. Cite the other bundle in
  prose instead.
- **Treating the gate's silence as correctness.** Gates verify structure, not sense. Links resolving
  and YAML parsing says nothing about whether a sentence contradicts the paragraph above it.

## Related Skills

- `okf-bundle` — stand up the bundle a concept lives in
- `okf-verify` — the cycle that re-checks these claims before they expire
- `diataxis-create` (diataxis-skills) — for prose whose job is teaching rather than reference;
  an OKF concept is closest to Diátaxis *reference* and *explanation*, and the two frameworks
  answer different questions about the same page
