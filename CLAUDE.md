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

- **The gate scripts.** They stay in `supplychain-workspace/workspace/scripts` — one copy, no drift.
  The skills ship self-contained checks that need only `python3` + `pyyaml`; they do not vendor the
  suite. A plugin makes gates travel for Claude Code users, which is not the same as making a
  published bundle verifiable by whoever fetches it. That gap is a standalone-binary problem.
- **RAG and retrieval.** OKF is a knowledge *format*; how a corpus is chunked, embedded and served is
  a different concern (see the `ragu-*` projects). See `Skills/CLAUDE.md` on Skills vs OKF vs RAG.
- **Diátaxis.** An OKF concept is closest to Diátaxis reference/explanation, but the frameworks
  answer different questions and neither subsumes the other. `diataxis-skills` stays separate.

## Development Conventions

- Skills run in the *target bundle's* working directory
- **Validation sections are executable** — commands with expected output, exit 0/1, usable as a hook
  or CI step unmodified. Per ADR-0009: prefer a check that fails over a paragraph that instructs
- **Do not ship a control you have not run.** Every check in this plugin was executed against a real
  bundle, and the structural ones were negative-tested against a deliberately broken fixture
- Checks depend on `python3` + `pyyaml` and nothing else. No plugin-local scripts to install
- Distinguish **spec** from **house convention** everywhere. OKF v0.2 requires parseable frontmatter
  and a non-empty `type`; almost everything else these skills recommend is a choice, and must be
  labelled as one
- SPDX license headers (MIT)
- `metadata.version` in every SKILL.md tracks `version` in `plugin.json` — bump in lockstep
- Installation: private incubator marketplace `jrjsmrtn-private-skills` (ADR-0005)

## Known Limits

- **n=2 bundles, one author.** The `stale_after` tier table and the type vocabulary are worked
  examples from one subject domain, not defaults. They are presented that way in the skills and
  should stay that way until a third bundle in an unrelated domain tests them.
- **`okf_version` targets OKF v0.2.** The spec-versus-convention table in `okf-concept` is pinned to
  that revision and needs re-checking when it moves.

## Remotes

- origin: private git server
- github: `jrjsmrtn/okf-skills` (private)
