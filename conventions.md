# L2 — Conventions

Judgment calls that no tool currently enforces. Each entry pairs the rule
with its negative space and, where possible, a way to check for violations.

## Source-string contract tests must not pin line layout (2026-09-02)

Contract tests that assert security or copy invariants by reading source
files (`readFileSync` + `toContain`) must pin identifiers, call arguments,
and copy — never exact line wrapping. The formatter re-wraps code and JSX
text freely, so exact-layout assertions break without any semantic change.

- Do: compare against whitespace-collapsed source:
  `src.replace(/\s+/g, ' ')`, and write regexes that tolerate
  formatter-added parentheses.
- Never: assert multi-line strings with hard-coded indentation.

## Comments explain why, with dates for scars (2026-09-02)

Comments state intent, invariants, and the reasoning a reader cannot
reconstruct from the code. Never narrate what the code plainly does.
Hard-won operational lessons carry a date (`(2026-09-02 incident)` style)
so future editors can trace context before removing the guardrail.

## Errors at API boundaries are stable codes, not messages (2026-09-02)

Route handlers and cross-process responses return stable snake_case error
codes (`payload_siteId_required`, `tenant_unbound`) that clients and tests
can pin; human-readable text may change freely. Interior code throws
`Error` with those same codes. Never let exception text become a client
contract, and never leak internals (SQL, stack, secrets) across a boundary.

## One validation library per project (2026-09-02)

A project picks exactly one schema/validation library (default: zod) and
uses it at every boundary. Two coexisting validation truths diverge
silently and double the type-identity hazard (a value parsed by library A
is not assignable to library B's output type). Third-party SDK type-compat
constraints can pin the exact version — record that pin where future
upgraders will see it.
