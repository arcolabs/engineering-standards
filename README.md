# Engineering Standards

The TypeScript engineering standard for Kai's projects (arcolabs and adjacent).
One truth source; projects adopt by vendoring a pinned snapshot.

## Layers

| Layer | Content | Lives in | Change cadence |
| --- | --- | --- | --- |
| L0 Principles | Judgment priors, rarely change | `principles.md` | Almost never |
| L1 Toolchain baseline | Executable config artifacts, not prose | `toolchain/` | Yearly, deliberate |
| L2 Conventions | Judgment calls machines cannot enforce | `conventions.md` | Quarterly, by friction |
| L3 Decisions | Dated ADRs with revisit conditions | `adr/` | Append-only |

Growth rule: a convention enters this repo only after real friction, ideally
observed in two projects (Rule of Three). Speculative standardization is a
liability, same as speculative abstraction.

## Adopting in a project

1. Vendor a snapshot of this repo into the project's `docs/engineering/`
   (or reference by tag) and record the adopted version tag in the project's
   `AGENTS.md`, e.g. `TS standard: arcolabs/engineering-standards@v0.1.0`.
2. Copy the L1 artifacts that apply (`toolchain/`) into the project root and
   wire the scripts/CI gates described in `toolchain/README.md`.
3. Project-specific rules stay in the project's own `AGENTS.md` / docs as a
   delta on top of this standard. This repo never absorbs project specifics.

## Changing the standard

- Changes land via PR with a date stamp; ADR entries are append-only
  (a reversed decision gets a new ADR superseding the old, never an edit).
- Every ADR must state its revisit condition: the evidence that would reopen
  the decision. A rule without a revisit condition becomes an untouchable
  fossil.
- When this standard and a machine-readable config disagree, the config is
  right and the doc is stale. Fix the doc.
