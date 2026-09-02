# ADR 0003 — Effect (effect.website): evaluated, deferred

- Status: deferred
- Date: 2026-09-02
- Supersedes: none

## Context

Effect is a mature, well-regarded TypeScript ecosystem: structured
concurrency, typed error channels, Layer-based DI, Schema, Stream, retry
and telemetry primitives. It is genuinely good engineering and worth
knowing.

## Decision

Effect is not part of the baseline and must not be introduced partially
into an existing project. A half adoption (only Schema, only Stream)
captures the costs and none of the compounding benefits.

## Rationale

- **It is a paradigm commitment, not a library.** The value proposition
  assumes everything flows through `Effect.gen`, Layer DI, and typed error
  channels. That is a rewrite of every module boundary — close to a
  one-way door for an existing codebase.
- **It solves no current bottleneck.** Our present pain is not in
  concurrency orchestration, cancellation, or error modeling. Paying a
  paradigm tax against a hypothetical bottleneck violates "dependencies are
  liabilities".
- **It collides with standing invariants.** Projects pin zod as the single
  validation library (in some repos an exact version is pinned for SDK type
  identity); Effect Schema would create a second validation truth.
- **Agent-authoring cost is real.** Correct Effect code requires holding
  the runtime model; fleet-written mediocre Effect fails more often than
  fleet-written mediocre plain TypeScript.

## Revisit when (all required)

1. A concrete subsystem's concurrency/retry/cancellation orchestration has
   outgrown hand-rolled control flow (candidate: employee runtime), AND
2. a time-boxed spike rewrites one real module in Effect with explicit
   acceptance criteria: readability by a cold reader, test cost, and agent
   rewrite success rate, AND
3. the zod single-validation-truth conflict has a resolution plan.

A greenfield service where Effect is the foundation from day one is a
legitimate separate experiment and does not require this ADR to be
reversed — but its result must be reported back here.
