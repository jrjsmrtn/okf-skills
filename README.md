<!--
SPDX-FileCopyrightText: 2026 Georges Martin <jrjsmrtn@gmail.com>
SPDX-License-Identifier: MIT
-->

# okf-skills

Claude Code plugin for authoring and maintaining **Open Knowledge Format** bundles.

OKF is a format for reference knowledge an agent can look things up in: one concept per Markdown
file, per-claim sources in frontmatter, and a stated expiry on every fact. It is what
[Agent Skills](https://code.claude.com/docs/en/skills) are not — a skill is a *procedure*, a bundle
is *what you consult while following one*.

## Skills

| Skill | Use when |
|---|---|
| `okf-bundle` | Creating a bundle, extracting one so it can be distributed, or auditing its structure |
| `okf-concept` | Adding or correcting a single concept — sourcing, attribution, `stale_after` |
| `okf-verify` | Running the review cycle — expiry triage, re-checking claims, version-currency sweep, release |

## What these encode

The three procedures were run by hand across two bundles before being written down. The parts worth
having a skill for are the ones that went wrong first:

- **The footnote label is a join key, not decoration.** A footnote whose label is not a
  `sources[].id` attributes nothing while looking like it does, and a footnote *definition* nobody
  references renders as nothing at all — a source that reads as cited is silently absent.
- **`verified` is an act.** A date moved to silence a warning is indistinguishable, to every gate
  ever written, from a claim that was actually re-checked.
- **"Is the claim accurate" and "is the cited version current" are different questions.** A full
  verification pass once confirmed every claim against a specification's pages — while that version
  had been retired.
- **A bundle that cannot leave its repository is not a bundle.** OKF calls it the unit of
  distribution; the version usually lives in a git tag, which stays behind when the bundle travels.
  The obvious fix — heading `log.md` by release — is a **spec violation** (§9 requires ISO 8601 date
  headings). The log is date-grouped by design, so a release map in the preamble is what is left.

## Validation

Every skill's Validation section is executable — commands with expected output, exit 0 on pass and
1 on failure, so they work unmodified as a hook or CI step
([ADR-0009](https://github.com/jrjsmrtn/project-orchestration-skills): prefer a check that fails over
a paragraph that instructs).

**Conformance is delegated to [`okfcli/okf`](https://github.com/okfcli/okf)**, the vendor-neutral Go
CLI — one static binary, JSON output, non-zero exit on error. It covers the spec more thoroughly than
a local script will: §5.1 footnote→`sources[].id`, §5.2 ISO datetimes, §5.5 expiry, §8 index
frontmatter, §9 log headings, and link resolution.

The skills add only what it does not check, measured against v0.2.1 on 2026-08-03:

- a footnote **cited but never defined** — renders as a dangling marker
- a footnote **defined but never cited** — renders as *nothing*, so a source that reads as cited is
  absent from the output
- `verified` present with no `sources` to have checked

Those need `python3` and `pyyaml`. Everything else, `okf` does better.

## Installation

Published to the private incubator marketplace
[`jrjsmrtn-private-skills`](https://github.com/jrjsmrtn/jrjsmrtn-private-skills).

```
/plugin marketplace add jrjsmrtn/jrjsmrtn-private-skills
/plugin install okf-skills@jrjsmrtn-private-skills
```

## Status

**0.1.0 — incubating.** Written from practice on two bundles, one author. That is enough to record
what was learned and not enough to call it general; treat the tier tables and type vocabularies as
worked examples rather than defaults.

## License

MIT
