# L0 — Principles

Stable judgment priors. Everything in L1–L3 should be derivable from these.
If a principle and a practice conflict, the practice is wrong or the
principle needs a dated amendment.

1. **Machine-enforceable never lives in prose.** If a tool can check a rule,
   the tool owns it. Documents hold judgment, rationale, and decision
   records only; a prose rule that a linter could enforce is process waste.
2. **Dependencies are liabilities.** Every dependency carries maintenance,
   security, and cognitive cost. It must pay rent against a current
   bottleneck, not a hypothetical one.
3. **Validate at system boundaries, trust inside.** Parse external input
   (HTTP, env, provider payloads, files) with a schema at the edge; interior
   code works with typed, already-valid values. Never validate the same fact
   twice at two layers.
4. **Evidence over assertion.** A bug gets a failing test before a fix; a
   performance claim gets a profile before an optimization. "No data found"
   is not "no data exists" — check existing logs and error fields before
   adding observability.
5. **Formatters own layout.** No human edits for style, no review comment
   about style, and no test that pins line wrapping. Layout is a solved,
   delegated problem.
6. **Types are contracts.** `strict` is the floor, not the ceiling. `any`
   may exist at a boundary only while being parsed away; it never crosses a
   module interface.
7. **Spec is the asset, code is consumable.** Invest in contracts, schemas,
   and decision records that survive a rewrite; keep implementation simple
   enough to regenerate from them.
8. **Order of operations: question → delete → simplify → speed up →
   automate.** Never accelerate or automate a step that has not been
   challenged, deleted, or simplified first.
9. **Decisions are dated and reversible-by-record.** Every standard carries
   its context and revisit condition. Understanding why a fence exists is a
   prerequisite for removing it.
10. **Agent-legible by construction.** Rules are greppable, pair positive
    form with explicit negative space ("do X, never Y"), and point to a
    deterministic check wherever one exists.
