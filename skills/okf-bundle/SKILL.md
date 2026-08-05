---
# SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
# SPDX-License-Identifier: MIT
name: okf-bundle
description: >
  Scaffold or restructure an Open Knowledge Format bundle — the directory layout, the reserved files and their constraints, link conventions, versioning, and gate wiring. Use when creating a new OKF bundle, extracting one from a parent repository, or auditing an existing bundle's structure.

metadata:
  version: "0.1.6"
license: MIT
---

# Scaffold an OKF Bundle

A bundle is a directory tree of Markdown concepts plus the metadata a consumer needs to judge them.
OKF calls it **the unit of distribution**, which is the property that should drive every structural
decision here: if it cannot leave the repository it was written in, it is not a bundle, it is a
`docs/` folder.

## When to Use

- Creating a new OKF bundle from nothing
- Extracting a bundle out of a parent repository so it can be distributed
- Auditing an existing bundle's structure, reserved files, or link conventions
- Deciding where a bundle's version lives

**Not for**: writing the concepts themselves (`okf-concept`) or running the review cycle
(`okf-verify`).

## Required Inputs

1. **Scope** — what the bundle is about, and explicitly what it is *not* about
2. **Distribution intent** — is publication the point, or is this private tracking? It changes
   licence, gate set and almost nothing else
3. **A place for the decisions** — bundles hold knowledge, not the reasoning about the corpus.
   Decide now where the scope boundary and admission test are recorded

## Layout

```
<bundle-repo>/
├── CLAUDE.md            # conventions: type vocabulary, stale_after tiers, link rules
├── CHANGELOG.md         # releases, Keep a Changelog
├── knowledge/           # THE BUNDLE — this is what gets distributed
│   ├── index.md         # bundle root: okf_version, then links out
│   ├── log.md           # content changes; ISO date headings (§9) + a release map in the preamble
│   ├── landscape.md     # optional: the read-straight-through explanation
│   └── <theme>/
│       ├── index.md     # subdirectory index — NO frontmatter
│       └── <concept>.md
└── .lefthook.yml        # gates
```

`knowledge/` is the bundle. Everything above it is repository furniture that a consumer copying the
bundle will not receive — which is the constraint that drives the versioning decision below.

## Reserved files and their constraints

`index.md` and `log.md` are reserved by the specification. The constraints on `index.md` are easy to
break and are not enforced by generic Markdown tooling:

- **`okf_version` is permitted only in the bundle-root `index.md`** (§8), and it declares which
  *specification* revision the bundle targets. It is not a content version.
- A **subdirectory** `index.md` takes no frontmatter at all.
- **`log.md` headings MUST be ISO 8601 dates** (§9) — see *Versioning* below, where this bites.
- An index's job is to link onward with a one-line gloss per entry. It is a table of contents, not a
  concept.

**No gate reads an `index.md` heading.** `okf validate` and `okf lint` both pass on a subdirectory
index whose title is wrong, misspelled or empty, and the link checker does not open `index.md` at all
— so a broken link *inside* an index is invisible to it too. These are the files where the tooling
stops helping, and the only ones that need reading with eyes before release.

Observed: four of six category indexes in a bundle shipped headed `# Uarchitecture`, `# Uinception`,
`# Ulearning`, `# Uorchestration`. Every gate passed. Only the two written by hand were correct.

**The cause is worth knowing, because it is not an OKF problem.** Those headings were generated with
`sed 's/^./\U&/'` to capitalise the directory name. **`\U` is a GNU sed extension**; BSD sed — the
default `sed` on macOS — has no case-conversion escape and emits the letter literally:

```bash
echo architecture | sed  's/^./\U&/'   # BSD  → Uarchitecture   ← silently wrong
echo architecture | gsed 's/^./\U&/'   # GNU  → Architecture
```

It fails plausibly rather than loudly, which is why it survived. When scaffolding index files, write
the headings by hand — there are six of them, once — or capitalise in the shell (`${name^}`, bash 4+)
rather than reaching for a `sed` escape whose availability varies by platform.

## Scope before content

Write the admission test before the first concept, and write it somewhere that is not the bundle.

A corpus grows by individually-defensible additions: each new subject is plausibly related, and the
result is a bundle nobody can keep current. The defence is a **stated test** that a candidate either
passes or fails, applied when the subject is proposed rather than after it is written.

A worked example, from a corpus about how bills of materials are obliged:

> *Does the instrument change what a bill of materials must contain, or when one must exist?*

That test excluded material that was genuinely interesting and genuinely adjacent. That is what a
test is for.

**Decisions about the corpus do not belong in the corpus.** Scope boundary, admission test, pruning
criterion and licence reasoning are decisions — put them in the governing project's decision log. The
bundle carries knowledge; a reader fetching it should not receive your ADRs.

## Links

- **Bundle-relative**, with a leading `/` meaning the *bundle root* — `/naming/purl.md` — not the
  filesystem root.
- **`/` does not cross bundle boundaries.** Two bundles sitting side by side on disk are still two
  units of distribution; a link from one into the other breaks the moment either is fetched alone.
  Cite the other bundle in prose, naming it.

## Versioning: the format fights you, and §9 is why

**OKF has no in-band content-version field.** `okf_version` is the specification revision, and it is
the only frontmatter an `index.md` may carry (§8) — so there is no conformant field for "which
release of this corpus is this".

A git tag does not solve it. A tag is a property of the *repository*; the bundle is `knowledge/`. A
tree copied into a container, vendored, or pasted into a context window arrives with no version at
all.

**Do not reach for a release heading in `log.md`.** It is the obvious move and the spec forbids it:

> §9: Date headings MUST use ISO 8601 `YYYY-MM-DD` form.

This was tried on two bundles on 2026-08-03, on the assumption that `log.md` bodies are unconstrained
prose. That assumption came from reading §5 and §8 and not reading §9. `okf validate` reported it as
six errors per bundle.

The deeper constraint is worth understanding rather than working around: **OKF's log model is
date-grouped, not release-grouped.** Five releases landing on one day share one heading and become
indistinguishable. Per-entry release attribution is not expressible in a conformant log — the format
has no concept of a release anywhere.

**What is conformant: a release map in the preamble**, which is unconstrained prose.

```markdown
# Bundle Update Log

**Releases**, newest first: **v0.5.0** 2026-08-02 · **v0.4.0** 2026-08-02 · **v0.1.0** 2026-08-01.
Unreleased work sits at the top of the newest date.

## 2026-08-03

* **Corrected**: …

## 2026-08-02

* **Added**: …
```

Update the map when cutting a release. `CHANGELOG.md` stays as the repository-level view.

Be honest about what this buys: a local convention that nothing validates and no consumer knows to
look for. It survives a copy. It is not interoperability.

### What the version means

SemVer's contract is about breaking a build; a knowledge bundle has no build. Define the surface
explicitly or the number carries nothing:

| | Event |
|---|---|
| **MAJOR** | a concept removed or moved (its path is its identifier), or **a claim reversed** |
| **MINOR** | concepts added, or one rewritten with its claims intact |
| **PATCH** | re-verification, added sources, expiry extension, prose fixes |

The reversal case is the one the number handles badly: a claim can flip at a stable path, which is
mechanically invisible. **The changelog carries what the number cannot** — write the reversal out,
name what it invalidates, and accept that the number is only an ordering.

## Gates

A bundle needs three checks, and one of them cannot be a hook:

| When | Checks | Catches |
|---|---|---|
| every commit | conformance, attribution, links, date format | anything a change breaks |
| **scheduled, unattended** | `stale_after` expiry | a corpus going stale with no commits |

**Expiry is a function of today's date, not of a diff.** A concept expires on a repository nobody is
committing to, so no commit hook will ever fire for it. Wire a scheduled run — `launchd`, `cron`, or
a CI schedule — and make its failure visible somewhere a human looks.

If the checkers live outside the bundle repository, **fail loudly when they are absent** rather than
skipping. A gate that quietly does nothing is worse than no gate: it reports success.

## Validation

**Use `okf` — do not hand-roll conformance.** The vendor-neutral Go CLI
([`okfcli/okf`](https://github.com/okfcli/okf)) validates the spec far more thoroughly than a local
script will: §5.2 ISO datetimes, §5.5 expiry, §8 index frontmatter, §9 log headings, and link
resolution. It is one static binary, emits JSON, and exits non-zero on error.

```bash
go install github.com/okfcli/okf/cmd/okf@latest   # or fetch a signed release + verify checksums

BUNDLE=knowledge
okf validate "$BUNDLE"    # exit 1 on any ERROR
okf lint "$BUNDLE"        # recommended fields; footnote -> sources[].id join (§5.1)
```

Expected on a clean bundle: `"errors": 0` and an empty `findings` array from both.

Then the release map, which is a local convention and therefore nobody else's job to check:

```bash
grep -qE '^\*\*Releases\*\*' "$BUNDLE/log.md" \
  && echo "release map present — a detached copy can name its version" \
  || echo "FAIL: no release map in log.md preamble"

# and confirm no heading violates §9, which is what a release heading would do
grep -nE '^## ' "$BUNDLE/log.md" | grep -vE '^[0-9]+:## [0-9]{4}-[0-9]{2}-[0-9]{2}$' \
  && echo "FAIL: non-ISO date heading above (OKF §9)" \
  || echo "all log.md headings are ISO dates"
```

## Anti-Patterns

- **A bundle that cannot leave its parent.** If `knowledge/` only makes sense inside the repository
  that holds it, it is documentation with extra frontmatter.
- **Decisions inside the corpus.** ADRs, scope arguments and licence reasoning ship to every consumer
  and answer questions they did not ask.
- **`okf_version` in a subdirectory index**, or any other frontmatter there (§8).
- **A release heading in `log.md`** — `## v0.5.0 — 2026-08-02`. §9 says date headings MUST be ISO
  8601 `YYYY-MM-DD`, so this is a spec violation however sensible it looks. Put the release map in
  the preamble.
- **Version in a tag only.** The tag stays behind when the bundle travels.
- **Hand-rolling conformance checks.** `okf validate` exists, covers more of the spec than a local
  script will, and is maintained by people reading the spec. Write local checks only for what it
  does not cover.
- **Adding a `docs/` tree.** Reference facts belong in the bundle; decisions belong in the governing
  project. A third location is where drift starts.
- **A gate that skips when its checker is missing.** Silence reads as success.

## Related Skills

- `okf-concept` — write the concepts this structure holds
- `okf-verify` — the cycle that keeps them true
- `bootstrap-project` (project-orchestration-skills) — tier, distribution profile and the repository
  furniture around the bundle
