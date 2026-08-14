---
# SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
# SPDX-License-Identifier: MIT
name: okf-verify
description: >
  Run the re-verification cycle over an OKF bundle — triage by expiry, re-check claims against upstream, run a version-currency sweep, prune what no longer earns its place, and stamp the release. Use when concepts are approaching `stale_after`, after a scheduled gate reports expiry, or before cutting a bundle release.

metadata:
  version: "0.1.8"
license: MIT
---

# Re-verify an OKF Bundle

A corpus with per-claim sources and expiry dates makes a promise: someone will come back and check.
This skill is that visit. It is the only routine in the set that is driven by the calendar rather
than by a change.

## When to Use

- Concepts are approaching or past `stale_after`
- A scheduled gate reported expiry
- Before cutting a bundle release
- After discovering that a cited specification moved

**Not for**: writing a new concept (`okf-concept`) or structural work (`okf-bundle`).

## Required Inputs

1. **The bundle root**
2. **A cycle boundary** — which concepts this pass covers. "All of them" is a valid answer only for
   a small corpus
3. **The bundle's tier table**, from its `CLAUDE.md`

## The rule the whole cycle rests on

**Never bump a `stale_after` without re-checking the claim.**

No gate can tell the difference between a date moved because the claim was verified and a date moved
because the warning was annoying. The second is worse than letting it expire: it converts a real
signal into a decorative one, permanently, and nothing downstream can detect it.

Corollary: **`verified` is an act, not a formatting step.** Absent is the honest default.

## Workflow

### Step 1 — Triage by expiry

```bash
BUNDLE=knowledge

python3 - "$BUNDLE" <<'PY'
import sys, yaml, pathlib, datetime
root = pathlib.Path(sys.argv[1]); today = datetime.date.today(); rows = []
for p in sorted(root.rglob('*.md')):
    if p.name in ('index.md', 'log.md'):
        continue
    meta = yaml.safe_load(p.read_text().split('---')[1]) or {}
    d = meta.get('stale_after')
    if isinstance(d, str):
        d = datetime.date.fromisoformat(d)
    rows.append(((d - today).days if d else 10**6, str(p.relative_to(root)), d,
                 bool(meta.get('verified'))))
for days, rel, d, v in sorted(rows):
    if days < 90:
        flag = "EXPIRED" if days < 0 else f"{days:>4}d"
        print(f"  {d}  {flag}  verified={'yes' if v else 'NO '}  {rel}")
print(f"  — {len(rows)} concepts, {sum(1 for r in rows if not r[3])} never verified")
PY
```

Work the list in date order. Concepts sharing a tier usually share sources, so they re-verify
together cheaply — that clustering is intended, not an accident to spread out.

### Step 2 — Re-check against upstream

For each concept in the pass, open the actual source and compare it to the claim. Not a search
summary of the source: **the source**.

Search summaries are wrong often enough to be treated as a lead rather than a finding. In one review
cycle they produced four separate errors that reading the primary document caught — a superseded
document cited as current, a contribution attributed to a paper that does not claim it, a pipeline
described with the wrong number of stages, and a figure that did not appear in the report it was
attributed to.

### When the primary source will not load

This is the step where a cycle quietly fails, because the fallback is so easy: the document does not
come back, a search result does, and the re-verification becomes the thing it exists to replace.

**Treat an unreachable primary source as a blocked check, not a downgraded one.** Either find another
route to the same document, or record in the concept that the claim rests on secondary sourcing —
never silently substitute.

The failure is rarely a clean error. Two shapes to recognise:

| Symptom | Why it is dangerous |
|---|---|
| **Success with no content** — HTTP 200/202, empty body | Nothing raises. A script reports "fetched", a reader assumes checked |
| **The wrong representation** — metadata, a landing page, an SPA shell | Large, plausible, and answers none of the questions you asked |
| **A bot challenge** — HTTP 200, a full HTML page, no document | Passes every check except "is this the thing I asked for" |

**A challenge page also destroys the existence signal.** Hosts running proof-of-work interstitials
return the same `200` for *any* path, including ones that do not exist — so on those hosts a status
code distinguishes neither retrieval nor existence, and *"source located, retrieval blocked"* and
*"the URL is wrong"* look identical. Two sources were once logged as blocked on that reasoning; both
URLs were simply wrong. **Confirm a URL from a page that actually resolves before recording a source
as unreachable.**

### When the source loads and does not support the claim

The previous section covers a source you cannot read. This one is more dangerous, because everything
succeeds: the document arrives, it is the right document, and **it does not say what the concept says
it says.**

The pull is to treat opening the source as the verification. Resist it. A source that loads and
supports nothing leaves the claim exactly where it was, and stamping `verified` because a fetch
returned 200 converts *unchecked* into *checked* without changing what is known.

**Record "checked and unsupported" as its own outcome.** Absent `verified` conflates two very
different states:

| State | What it means | What it needs next |
|---|---|---|
| never checked | nobody has looked | someone looks |
| **checked, unsupported** | someone looked; this source does not carry the claim | a *different* kind of evidence |

Only the second tells the next person not to repeat the same lookup. Write it into the concept — what
was read, which single sentence it did support, and what it was silent on.

Three routes out, in the order worth trying:

1. **A different source of the same kind** — a specification that states an obligation rather than a
   description that states an intent.
2. **A demonstration.** Where a claim is behavioural, the honest form is reproducible on demand
   against a named tool and version, not a citation.
3. **Demote the claim in place.** Mark it as reasoning rather than sourced fact and leave it visible.
   A concept that says which of its sentences are load-bearing and which are inference is more useful
   than one that hides the difference.

**One heuristic, from doing this to three concepts at once.** Project documentation describes
*purpose*, not failure modes. A tool's own page will tell you precisely what a mechanism is *for* and
be nearly silent on what goes wrong with it — including the failure everyone actually discusses. One
concept asserted a well-known failure mode and cited the vendor page for it; the page never mentions
it. **A tool's documentation is evidence about its documentation.** Failure modes have to come from
specifications, incident writeups, or measurement.

Removing a citation is a legitimate outcome of this step. A source retained because it looks
supporting, when it is not, is worse than no source: it is what a reader checks least.

### When the primary source is a scan

A PDF with no text layer is not unreachable — it is unread. `pdftotext` and similar return **exit 0
and an empty file**, which a script reports as success. Check the **word count**, never the status.

Where a text layer is absent, OCR is the route (`xberg` and similar run it automatically). Two rules
make the difference between a usable source and a corrupted record:

- **Record which path produced the text.** Extractors expose this — `extraction_method` is `native`
  for a text layer, `ocr` for recognition. Note it in the re-verification notes; the next reviewer
  needs to know how much to trust the transcription.
- **OCR output is not quotable without verification.** Recognition silently substitutes characters,
  and a wrong character *inside quotation marks* is worse than no quote: it is citable, it looks
  authoritative, and nothing downstream will catch it. **Before quoting an OCR'd passage, check it
  against the rendered page.**

This is the sharpest form of the corpus's central risk. A concept whose claim is merely stale is
visibly stale; a concept whose *quotation* is an OCR artefact reads as perfectly sourced and is
wrong in the one place the format promises accuracy.

### When one source publishes two versions

A specification, policy or standard often exists as HTML *and* PDF, or as a rendered page *and* a
repository. **They can disagree, and then the comparison is the verification** — reading either alone
returns a confident wrong answer.

Observed: a vendor's open-source policy prohibits a practice in its dated PDF and says nothing about
it on the live web page. Which is current was not establishable, and the widely-repeated claim about
that vendor traces to the PDF alone.

Two techniques:

- **Count, do not skim.** Extract the text and count occurrences of the term. "I did not see it" is
  not a finding.
- **Prove absence rather than truncation.** Check for the sections that *bracket* the passage you
  expect. If the neighbours are present and the passage is not, it is genuinely absent — otherwise
  you may be reading a partial fetch.

When the two disagree and nothing settles which governs, **record the disagreement as the finding**
and cite both. Picking one silently is the failure.

Worked example, found on 2026-08-03 re-verifying an EU regulation. `eur-lex.europa.eu` returns
**HTTP 202 with a zero-byte body** to every non-browser client — agent fetch tools and `curl` alike,
on the ELI, HTML and PDF endpoints. The working route was content negotiation on a different host:

```bash
curl -sL -H "Accept: application/xhtml+xml" -H "Accept-Language: eng" \
  "https://publications.europa.eu/resource/celex/32024R2847" -o doc.xhtml
```

Both headers are load-bearing — `Accept` alone returns 400, and with no `Accept` at all the same URL
serves RDF metadata that answers no legal question.

**Record the route when it is non-obvious**, in the project's conventions rather than in the concept:
a concept carries the claim, not the retrieval recipe. The next reviewer should not have to rediscover
it, and "the source was unreachable" should appear in the log rather than being papered over.

**Verify the recipe before you write it down.** The command above was first recorded from memory,
minus `Accept-Language`, and returned 400 for every document tried. A retrieval method is a control;
an unrun one is a claim.

When a claim changes:

- Correct the concept, and **name what the old claim would have misled someone into believing**. A
  reader who cited the old text needs to know what they now have wrong.
- Repoint every concept that cited it. A taxonomy that reassigns its own labels silently invalidates
  every cross-reference by label.
- Add the `verified` entry.

When a claim holds: add the `verified` entry, extend `stale_after` by its tier, and say so in the
log. **A negative result is a result** — record it, dated, so the next cycle does not redo it blind.

### Step 3 — Run a version-currency sweep

This is a **different question** from "is this claim accurate", and asking only the first is how a
corpus goes stale while every concept reads as verified.

> For every concept citing a versioned specification: *is the cited version still the current one?*

The distinction is not academic. In the reference corpus a full verification pass checked claims
against a specification's v1.1 pages and confirmed them correct — while v1.1 had been retired. Every
individual claim was accurate about a superseded document. The error was found six months later, by
accident, on unrelated work.

Run the sweep as its own pass, and record the negative result with the list of what was confirmed
current.

### Step 4 — Re-tier what was misfiled

If a concept expired faster than its tier predicted, the tier was wrong. Move it, and record *why*
the volatility class was misjudged.

Moving a date for this reason is **not** verification and earns no `verified` entry. Keeping those
two acts distinct is what lets the field mean something.

### Step 5 — Prune

A tracking corpus needs a deletion criterion or it only grows. A workable one:

> Past `stale_after` for two consecutive review cycles → **delete, don't carry**.

This requires countable cycles, which is why the bundle's releases are tagged and its `log.md`
preamble carries a release map. Deletion is a legitimate outcome of a review, and a concept nobody re-verified
twice is telling you it was not worth the file.

### Step 6 — Stamp the release

1. Entries accumulated under the newest ISO date heading in `knowledge/log.md`
2. Add the release to the **preamble release map** — `**v0.6.0** 2026-08-14 · **v0.5.0** …`
3. Mirror it in `CHANGELOG.md`
4. **Re-read `index.md` and any `landscape.md`** against what actually changed
5. Tag

**Step 4 exists because a verification pass is precisely when a summary goes stale.** You edit
concepts; the index describes concepts; nothing links the two. An index has been observed carrying
*"Nothing here is verified"* through two tagged releases after a pass verified three of its concepts,
and a `landscape.md` describing a concept as *"the concept that failed its verification"* after that
concept had been verified. Neither is detectable by any gate — see `okf-bundle` for why an index that
characterises a corpus holds a second copy of it.

**Do not rename a heading to the release.** `## v0.6.0 — 2026-08-14` violates OKF §9, which requires
date headings in ISO 8601 `YYYY-MM-DD` form. The log is date-grouped by design; the release map in
the preamble is the conformant place a version lives, and `okf validate` will fail the other form.

**Lead the changelog entry with what was learned, not what was touched.** A file list is recoverable
from git; the synthesis is not. If three corrections in a release converge on one structural point,
say the point.

## Validation

Everything below must pass before the tag.

```bash
BUNDLE=knowledge

# 1. Spec conformance, including expiry (§5.5) and log headings (§9)
okf validate "$BUNDLE"   # exit 1 on any ERROR; stale concepts appear as WARN findings
okf lint "$BUNDLE"

# 2. Nothing claims verification it cannot have had — okf does not check this
python3 - "$BUNDLE" <<'PY'
import sys, yaml, pathlib
root = pathlib.Path(sys.argv[1]); fail = False
for p in sorted(root.rglob('*.md')):
    if p.name in ('index.md', 'log.md'):
        continue
    meta = yaml.safe_load(p.read_text().split('---')[1]) or {}
    if meta.get('verified') and not meta.get('sources'):
        print(f"FAIL {p.relative_to(root)}: `verified` with no `sources` to have checked")
        fail = True
if not fail:
    print("verification coherence OK")
sys.exit(1 if fail else 0)
PY

# 3. The release is named inside the bundle, not only in a git tag
grep -qE '^\*\*Releases\*\*' "$BUNDLE/log.md" \
  && echo "release map present in the preamble" \
  || echo "FAIL: no release map — a detached copy cannot name its version"

# 4. No heading violates §9 (a release heading is the way this goes wrong)
grep -nE '^## ' "$BUNDLE/log.md" | grep -vE '^[0-9]+:## [0-9]{4}-[0-9]{2}-[0-9]{2}$' \
  && echo "FAIL: non-ISO date heading above (OKF §9)" \
  || echo "all log.md headings are ISO dates"
```

To triage expiry *before* it fails, use Step 1's listing — `okf validate` tells you what is already
stale, not what is about to be.

## Anti-Patterns

- **Bumping the date to silence the warning.** The one failure the tooling cannot see.
- **Stamping a tier.** "Reviewed the ~6 month tier" is not "checked this concept". Stamp what you
  actually opened, and record which ones you left.
- **Verifying claims without checking the version.** Every claim can be accurate about a superseded
  document.
- **Trusting a search summary as a source.** Treat it as a lead. Open the document.
- **Letting an unreachable source become a secondary one.** The substitution is silent and the
  `verified` entry looks identical afterwards. Say the source was unreachable, or find another route.
- **Treating a 200 as a fetch.** An empty body, a landing page or a metadata representation all
  return success. Check that what came back is the document.
- **Correcting a claim without repointing its citers.** A reassigned label leaves every
  cross-reference quietly wrong.
- **Letting the corpus only grow.** Without a deletion criterion, review becomes archiving.
- **A changelog that lists files.** Git already has that. Write what changed in what you know.

## Related Skills

- `okf-concept` — how a claim gets sourced and tiered in the first place
- `okf-bundle` — the scheduled gate wiring that makes expiry visible at all
- `release-prep` (project-maintenance-skills) — the surrounding release mechanics
