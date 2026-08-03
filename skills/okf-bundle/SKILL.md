---
# SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
# SPDX-License-Identifier: MIT
name: okf-bundle
description: >
  Scaffold or restructure an Open Knowledge Format bundle — the directory layout,
  the reserved files and their constraints, link conventions, versioning, and gate
  wiring. Use when creating a new OKF bundle, extracting one from a parent
  repository, or auditing an existing bundle's structure.
metadata:
  version: "0.1.0"
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
│   ├── log.md           # content changes, headed by release
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

- **`okf_version` is permitted only in the bundle-root `index.md`**, and it declares which
  *specification* revision the bundle targets. It is not a content version.
- A **subdirectory** `index.md` takes no frontmatter at all.
- An index's job is to link onward with a one-line gloss per entry. It is a table of contents, not a
  concept.

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

## Versioning: put it inside the bundle

**OKF has no in-band content-version field.** `okf_version` is the specification revision, and it is
the only frontmatter an `index.md` may carry — so there is no conformant field for "which release of
this corpus is this".

A git tag does not solve it. A tag is a property of the *repository*; the bundle is `knowledge/`. A
tree copied into a container, vendored, or pasted into a context window arrives with no version at
all.

**Head `log.md` by release.** Its body is free prose, so this is conformant, and it is the only place
a version survives detachment:

```markdown
## Unreleased

* **Added**: …

## v0.5.0 — 2026-08-02

* **Corrected**: …
```

New entries land under `## Unreleased`; cutting `vX.Y.Z` renames that heading and adds the ISO date.
`CHANGELOG.md` stays as the repository-level view of the same releases.

Be honest about what this buys: it is a local convention that nothing validates and no consumer knows
to look for. It survives a copy. It is not interoperability.

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

```bash
BUNDLE=knowledge   # the bundle root

python3 - "$BUNDLE" <<'PY'
import sys, yaml, pathlib
root = pathlib.Path(sys.argv[1])
fail = False

if not (root / 'index.md').exists():
    print(f"FAIL: no {root}/index.md — a bundle needs a root index"); fail = True

n = 0
for p in sorted(root.rglob('index.md')):
    n += 1
    src = p.read_text()
    if not src.startswith('---'):
        continue
    meta = yaml.safe_load(src.split('---')[1]) or {}
    allowed = {'okf_version'} if p.parent == root else set()
    extra = set(meta) - allowed
    if extra:
        where = "the bundle root index" if p.parent == root else "a subdirectory index"
        print(f"FAIL {p}: {where} may not carry {sorted(extra)}")
        fail = True

concepts = [p for p in root.rglob('*.md') if p.name not in ('index.md', 'log.md')]
missing = [p for p in concepts
           if not (yaml.safe_load(p.read_text().split('---')[1]) or {}).get('type')]
for p in missing:
    print(f"FAIL {p}: no `type` — the one key OKF requires"); fail = True

if not fail:
    print(f"structure OK ({n} indexes, {len(concepts)} concepts)")
sys.exit(1 if fail else 0)
PY
```

Then confirm the version actually lives inside the bundle:

```bash
grep -qE '^## (v[0-9]+\.[0-9]+\.[0-9]+|Unreleased)' "$BUNDLE/log.md" \
  && echo "log.md is headed by release" \
  || echo "FAIL: log.md carries no release heading — a detached copy has no version"
```

## Anti-Patterns

- **A bundle that cannot leave its parent.** If `knowledge/` only makes sense inside the repository
  that holds it, it is documentation with extra frontmatter.
- **Decisions inside the corpus.** ADRs, scope arguments and licence reasoning ship to every consumer
  and answer questions they did not ask.
- **`okf_version` in a subdirectory index**, or any other frontmatter there.
- **Version in a tag only.** The tag stays behind when the bundle travels.
- **Adding a `docs/` tree.** Reference facts belong in the bundle; decisions belong in the governing
  project. A third location is where drift starts.
- **A gate that skips when its checker is missing.** Silence reads as success.

## Related Skills

- `okf-concept` — write the concepts this structure holds
- `okf-verify` — the cycle that keeps them true
- `bootstrap-project` (project-orchestration-skills) — tier, distribution profile and the repository
  furniture around the bundle
