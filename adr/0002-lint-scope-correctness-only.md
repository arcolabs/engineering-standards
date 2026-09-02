# ADR 0002 — Lint scope: correctness gates only, style is the formatter's

- Status: accepted
- Date: 2026-09-02
- Supersedes: none

## Context

Lint configs tend to accrete: presets get enabled wholesale, style rules
duplicate the formatter's job, and the signal-to-noise ratio collapses
until developers ignore or mass-disable warnings. arcops runs ESLint 9 +
typescript-eslint with every general preset deliberately disabled; its only
active rule is a project-specific security gate (tenant raw-db import ban
with a shrink-only baseline ratchet).

## Decision

1. The formatter owns 100% of style. Lint configs contain zero style rules.
2. Lint exists for two purposes only:
   - compiler-adjacent correctness the typechecker cannot express, and
   - project-specific security/architecture gates (import bans, forbidden
     APIs), ideally enforced as dedicated CI jobs with ratchets.
3. New general-purpose rule presets are OFF by default. Enabling one
   requires naming the concrete bug class it would have caught in our own
   history.
4. oxlint is the designated candidate for (2)-style correctness rules at
   near-zero CI cost, but is not adopted yet — adopting a linter to have a
   linter is exactly the accretion this ADR exists to prevent.

## Alternatives considered

- **ESLint with typescript-eslint recommended presets**: rejected. Broad
  presets optimize for rule coverage, not for our defect history; the cost
  is triage noise and rule-by-rule appeasement edits.
- **oxlint replacing ESLint wholesale**: rejected for now. Replacing a
  working, CI-enforced security gate for zero user-visible benefit trades
  real equivalence risk for vanity uniformity.

## Revisit when

- A concrete correctness bug class recurs that `strict` tsc does not catch
  (evaluate oxlint against that class specifically), or
- oxlint reaches stable parity for the project-specific gates a repo
  actually depends on, making one-tool consolidation a simplification
  rather than a swap.
