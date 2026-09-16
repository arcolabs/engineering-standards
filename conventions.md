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

## Frontend changes route to evidence by what the diff touches (2026-09-16)

This is the project's Definition of Done for frontend work — the Scrum term is
used deliberately rather than a local name, because the artifact is exactly that:
what must be true before anyone may say a change is finished.

A change is not ready until it carries the evidence its own diff requires.
"Verified" without the artifact named below means not verified: provenance of
evidence outranks its content, and a self-report is not evidence (2026-07-23).

Route on the first rule that matches:

1. **Non-UI logic only** — the gate command plus its real output, or the
   assertion that fails before the fix and passes after. Never a bare "passed".
2. **Rendered output** (markup, style, layout) — the affected routes at both
   desktop and mobile sizes, plus console error count. Here a screenshot is
   valid evidence, and it is valid for *this* claim only.
3. **Behavior** (handlers, state machines, media, async transitions) — an
   assertion on the end state after the interaction: a scripted check, or a
   recorded trace. A static screenshot never suffices; posters painting is not
   video playing (2026-08-10).
4. **Build, deploy, cache, or config** — read the deployed build identity
   *before* diagnosing the page, then one named observable with its pre-change
   value and the end time of the observation window. A defect that heals on the
   next build was never a defect (2026-09-12).
5. **In any repo, when the gate being cited does not run in CI** — no
   independent evidence exists. Wiring the workflow is part of "done", not a
   follow-up (2026-07-23).

Rules 1–4 answer "what do I owe". They are ordered by blast radius, not by
effort, and an agent should classify the diff, not the ticket.

Ownership boundary: this entry owns **evidence type** — what a claim of "verified"
must deliver. **Cadence** is out of scope for a project standard on purpose: how
much verification to spend mid-slice, and when to batch small edits into one gate
run, is a working rhythm of whoever is doing the work, not a property of the code.
Silence here means out of scope, not permission to verify per keystroke.

## Browser test scope: only what we control, and no test-shape quotas (2026-09-16)

Third-party responses are stubbed at the network layer, never driven live
(`page.route()` and equivalents): an external page's cookie banner, rate limit,
or redesign makes a test fail for reasons that are not our defect. Note the
boundary of that rule — it is a *reliability* argument about flake, not a
license to leave a paid path unverified. Whether a real outward action ever gets
automated is decided by side-effect governance and deliberate quota allocation,
never by citing a flake rule.

- Never: gate on code-coverage percentage. It measures that a line ran, not what
  it did, and any stated threshold gets treated as a ceiling rather than a floor.
- Never: adopt the 80/15/5 test pyramid as a requirement. The frontend
  counterpart is the Testing Trophy (static / unit / integration / end-to-end);
  keep the pyramid's *warning* — an ice-cream-cone suite of many e2e tests and
  few others is slow, unreliable, and hard to debug.
- Never: claim a mix ratio from industry shape when local failure data exists.
  The dated incidents behind rules 1–5 are CI-absence, evidence-type, and
  build-identity failures: zero of three were test-shape failures, so no mix
  claim earns a place in this standard until a measured failure asks for one.
- Never: invent a tier vocabulary for verification. Release risk is already
  tiered (T0/T1/T2) by the owner of that policy; reuse it or say nothing.
- Do: prefer user-facing locators (`getByRole`, accessible name) over CSS
  classes and XPath, because DOM structure changes for reasons unrelated to
  behavior. This makes the existing accessibility floor pay for itself on DOM
  surfaces; it does not apply to canvas or native shells, where readable
  internal state is the equivalent hook.

## Red gates block by default; only the owner overrides, per instance (2026-09-16)

A failing gate stops the merge. This standard never self-authorizes an override:
an exception is the human owner's decision, taken per instance and recorded in
the commit message or issue that ships on the exception.

- Never: retry a red gate into green and stay silent about the retry count.
- Never: quarantine a flaky test permanently. Flaky with the underlying path
  unchanged is a bad test: fix or delete it in the same change, and write down
  which.
- Never: put credentials for flows under browser test into a repo, a trace, or a
  CI artifact. Test accounts only; a saved storage state is a secret and is
  treated as one.
