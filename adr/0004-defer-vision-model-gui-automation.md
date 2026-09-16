# ADR 0004 — Defer vision-model GUI automation; add no browser tooling to L1

- Status: accepted (Kai, 2026-09-16)
- Date: 2026-09-16
- Supersedes: none

## Context

`midscenejs.com` was evaluated as a candidate for UI verification. It is an
MIT-licensed GUI agent that drives interfaces from screenshots via a
multimodal model, so tests are written as natural-language intent rather than
selectors. Its stated headline capability is reaching elements that selectors
cannot: unlabeled controls, canvas, native apps, cross-origin frames.

Two things had to be settled separately: whether our surfaces actually need
that capability, and whether our CI can run browser automation at all today.

Measured state (2026-09-16, re-checked by an independent review pass):

- **Selector-opaque surfaces we own:** pngtubermaker's canvas components
  (`PNGTuberPreview.tsx`, `PlayerClient.tsx`, `widget/FloatingAvatar.tsx`,
  `lib/pngtuber-engine.ts`) and PngTuber's Electron desktop app (electron 39 +
  electron-vite). Everything else — arco, arcops-server, marketing surfaces,
  auth, billing — is ordinary DOM.
- **Zero browser tooling exists anywhere:** no `@playwright/test`, no
  `@testing-library/*`, no `vitest` in any of the four product repos.
- **CI capacity is the binding constraint, not tooling.** `arco` and
  `arcops-server` share one self-hosted runner (label `arcops`, on NAB9 since
  2026-09-15). GitHub-hosted minutes are blocked by an org billing failure
  recorded in `arco/.github/workflows/ci.yml`; pngtubermaker's `pr.yml` still
  declares `ubuntu-latest` but has no run since 2026-09-11. Observed same-day
  on the shared runner: a deterministic gate job in 24 seconds, then two docker
  image jobs at 27 and 18 minutes, both failing, one run wall-clocking 2h07m.
  PngTuber has no workflow at all.

So the honest reason not to adopt browser automation right now is not
classification theory: it is that the two most mature repos cannot host it, and
the runner that would have to absorb it is currently failing on docker builds.

## Decision

1. **No browser-automation tooling in L1.** No `playwright.config.base.ts` and
   no equivalent artifact. This repo's growth rule requires friction observed in
   two projects before a convention enters; a first-party L1 artifact with zero
   adopters is the speculative standardization that rule exists to prevent. The
   config earns its place by landing with the first repo that adopts it.
2. **Vision-model GUI automation is deferred, not rejected**, with the revisit
   conditions below. It follows ADR 0003's precedent (Effect deferred, all
   conditions required).
3. **What to test before what to test with.** Evidence routing and test-scope
   rules live in `../conventions.md`; they are tool-independent and are the part
   the dated incidents actually justify.

## Why deferral rather than rejection

- Its differentiating claim does not apply to our strongest candidate. Playwright
  reaches an Electron renderer as a real `Page`, and its `codegen` + role-based
  locators solve the DOM cases. Caveat recorded so nobody assumes it is free:
  Playwright's Electron support is documented **experimental**, uses the
  `_electron` namespace, and does not intercept native dialogs — `showOpenDialog`
  / `showSaveDialog` / `showMessageBox` must be stubbed through
  `electronApplication.evaluate()` for tests to be deterministic.
- For canvas the real problem is content assertion, not element location. The
  first move is exposing readable engine state from `pngtuber-engine.ts`; pixel
  baselines are the fallback. This is the load-bearing experiment that keeps the
  deferral honest — if it fails on a paid path, condition 2 below is met.
- It duplicates an owned lane. Browser control for interactive verification
  already has a named owner: the machine's managed browser lane (agent-browser +
  chromium-hl), governed by the exogenous-resource rules.
- It converts a regression gate from free-to-rerun into billed-per-execution at
  every step, and the interactive-only subscription lanes cannot host unattended
  CI. That is a new recurring spend requiring deliberate allocation, and the
  current bottleneck is not located there.

## Revisit when

All conditions are stated as evidence that would reopen this, per the repo's
rule that a decision without a revisit condition becomes a fossil.

1. A **native mobile or HarmonyOS** surface enters the product line. This is the
   capability with no Playwright substitute; the argument becomes real rather
   than borrowed.
2. A **canvas or otherwise selector-opaque path becomes paid AND stable AND
   CI-worthy**, and exposing readable internal state for it has actually been
   attempted and failed.
3. A deterministic browser suite exists and **authoring cost** is the measured
   bottleneck. Then evaluate AI test-authoring, including this tool class in
   author-time mode, where model spend happens once to produce test code we own
   rather than on every CI run.
4. CI capacity exists: a runner can host a browser job inside the ten-minute
   feedback budget without the docker-build contention pattern observed
   2026-09-16.

## Out of scope for this record

Whether a real outward action (e.g. an actual publish to a third-party
platform) is ever automated is **not** decided here. That is a
side-effect-governance call for the human owner, made per instance with the
spend deliberately allocated. Reliability arguments about flake are not
authority for it in either direction.

## Alternatives considered

- **Adopt now as the CI e2e layer.** Rejected on the measured facts above.
- **Add the L1 config artifact now "so it's ready".** Rejected: zero adopters,
  and it would have been the second time this ADR's subject repo published an
  unfrictioned standard.
- **Formalize the current practice without any tooling commitment.** Rejected for
  the tooling parts only; the evidence parts of this option are exactly what
  `../conventions.md` now does, which is why this ADR carries no policy.
