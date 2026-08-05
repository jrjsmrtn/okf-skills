# OKF Skills

Claude Code plugin providing skills for authoring and maintaining **Open Knowledge Format** bundles.

## Overview

3 skills covering the lifecycle of a knowledge bundle: scaffold it (`okf-bundle`), write a concept in
it (`okf-concept`), keep it true (`okf-verify`).

This is a **format plugin**, in the family of `diataxis-skills`, `c4-skills` and `obsidian-skills` —
one artifact format, its conventions and its checks. It is not a lifecycle plugin; ADR-0008's
orchestration/maintenance taxonomy governs that other family and does not apply here.

## Provenance

The procedures were extracted from practice on two OKF bundles in the `SupplyChain` workspace
(`software-supply-chain-landscape`, 65 concepts, and `instruction-artifact-supply-chain`), after each
had been run by hand at least twice. Nothing here is speculative generalisation from the
specification — every rule in the skills exists because doing it the other way went wrong first.

Both bundles' conventions were, at the time of extraction, duplicated across two `CLAUDE.md` files
and edited in lockstep. That duplication is what the plugin replaces.

## Scope Boundary

**In**: the OKF format, concept authoring, attribution mechanics, expiry and re-verification, bundle
structure and versioning.

**Out**:

- **Conformance checking.** Delegated to [`okfcli/okf`](https://github.com/okfcli/okf), the
  vendor-neutral Go CLI. It covers §5.1/§5.2/§5.5/§8/§9 and links — more of the spec than the local
  gates ever did, and it is maintained by people reading the spec. The skills add only the three
  residual checks it does not perform (both footnote-*definition* directions, and `verified` without
  `sources`). Do not reimplement conformance here.
- **The local gate scripts.** They stay in `supplychain-workspace/workspace/scripts`. As of
  2026-08-03 two of the three are superseded by `okf validate`; retiring them is tracked in that
  workspace's ADR-0010, not here.
- **RAG and retrieval.** OKF is a knowledge *format*; how a corpus is chunked, embedded and served is
  a different concern (see the `ragu-*` projects). See `Skills/CLAUDE.md` on Skills vs OKF vs RAG.
- **Diátaxis.** An OKF concept is closest to Diátaxis reference/explanation, but the frameworks
  answer different questions and neither subsumes the other. `diataxis-skills` stays separate.

## Development Conventions

- Skills run in the *target bundle's* working directory
- **Validation sections are executable** — commands with expected output, exit 0/1, usable as a hook
  or CI step unmodified. Per ADR-0009: prefer a check that fails over a paragraph that instructs
- **Do not ship a control you have not run.** Every check in this plugin was executed against a real
  bundle, and the structural ones were negative-tested against a deliberately broken fixture. This
  is not ceremony: 0.1.0 shipped versioning guidance that violated OKF §9, and running `okf validate`
  is what caught it
- **Claims about what `okf` does or does not check must be re-measured, not remembered.** The
  coverage table in `okf-concept` is pinned to v0.2.1 (2026-08-03); upstream is active
- Residual checks depend on `python3` + `pyyaml` and nothing else. No plugin-local scripts to install
- Distinguish **spec** from **house convention** everywhere. OKF v0.2 requires parseable frontmatter
  and a non-empty `type`; almost everything else these skills recommend is a choice, and must be
  labelled as one
- SPDX license headers (MIT)
- `metadata.version` in every SKILL.md tracks `version` in `plugin.json` — bump in lockstep
- Installation: public marketplace `jrjsmrtn-skills` (ADR-0005); graduated 2026-08-05 per skills-workspace ADR-0012 as amended

## Known Limits

- **n=2 bundles, one author.** The `stale_after` tier table and the type vocabulary are worked
  examples from one subject domain, not defaults. They are presented that way in the skills and
  should stay that way until a third bundle in an unrelated domain tests them.
- **`okf_version` targets OKF v0.2.** The spec-versus-convention table in `okf-concept` is pinned to
  that revision and needs re-checking when it moves.

## Remotes

- origin: private git server
- github: `jrjsmrtn/okf-skills` (private)
