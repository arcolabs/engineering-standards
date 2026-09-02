# ADR 0001 — oxfmt as the formatter

- Status: accepted
- Date: 2026-09-02
- Supersedes: none

## Context

Projects had no formatter at all; style was maintained by hand and drifted
(arcops: 3903 single-quote imports vs 1 double-quote before normalization).
A formatter is the cheapest correctness-adjacent win: it deletes a whole
class of review noise and makes layout a delegated, solved problem.

## Decision

oxfmt (oxc project) is the standard formatter. Baseline config:
`toolchain/.oxfmtrc.json` (single quotes, 2-space indent, width 100,
trailing commas, LF).

## Alternatives considered

- **Prettier**: the most proven option and oxfmt's output target. Rejected
  not on capability but on direction: the oxc/VoidZero stack is where the
  Vite ecosystem is investing, and our projects are Vite/Bun-native. Speed
  was explicitly NOT a deciding factor — at our repo sizes both finish in
  ~1s; anyone re-arguing this ADR on benchmark numbers is relitigating a
  non-bottleneck.
- **Biome**: capable unified formatter+linter, but its config model
  diverges from Prettier semantics, and dedicated tools (oxfmt + a
  separately-chosen linter) keep each decision independently reversible.

## Consequences

- oxfmt is pre-1.0. Known rough edge (0.66): not one-pass idempotent on
  functions with multi-line object-type return annotations; the `format`
  script runs it twice to reach the fixed point. Delete the second run once
  upstream converges in one pass.
- Tests must never pin line layout (see conventions.md).

## Revisit when

- oxfmt issue tracker shows recurring check-flap/idempotency bugs after
  1.0, or the project stalls — then fall back to Prettier (output is
  compatible by design, so the fallback is one config swap plus one
  reformat commit).
