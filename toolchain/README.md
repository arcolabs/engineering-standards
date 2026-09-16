# L1 — Toolchain baseline

Executable artifacts, not descriptions. Copy the ones that apply into the
project root and wire the gates below. The decision rationale for each
choice lives in `../adr/`; this file only covers wiring.

## Artifacts

| File | Purpose | Install |
| --- | --- | --- |
| `.oxfmtrc.json` | Formatter, the sole owner of layout | `bun add -d oxfmt` |
| `tsconfig.base.json` | Strictness floor | `"extends": "./tsconfig.base.json"` in the project tsconfig |

## Runtime and package manager

Bun-first (`packageManager` pinned in package.json). Single lockfile at the
repo root; workspaces never carry their own lockfile.

## Scripts (package.json)

```json
{
  "scripts": {
    "format": "oxfmt && oxfmt",
    "format:check": "oxfmt --check",
    "typecheck": "tsc --noEmit",
    "ci": "bun run format:check && bun run typecheck && bun run lint && bun run test && bun run build"
  }
}
```

`format` runs oxfmt twice on purpose: oxfmt (as of 0.66) is not one-pass
idempotent on functions with multi-line object-type return annotations —
pass one re-wraps, pass two reaches the fixed point
([oxc-project/oxc#26240](https://github.com/oxc-project/oxc/issues/26240)).
A single-pass format followed by CI `format:check` can flap on that
pattern. Recheck when upgrading oxfmt; delete the second run once upstream
is idempotent.

## CI gates

Every PR and every push to main runs, as separate jobs: `format:check`,
`typecheck`, `lint`, `test`, `build`. A green merge button requires all of
them. Gates are deterministic and hermetic: no network-dependent checks in
the required path.

Two constraints on the gate list itself:

- **Budget:** the full gate set for one change must return feedback in under
  ten minutes, locally and on CI. Over budget means pruning the suite, not
  raising the budget.
- **Integrity:** a gate named in a `ci` script chain and a job CI actually runs
  are different facts. When they disagree, the workflow is the truth and the
  script line is drift (found 2026-09-16 in an adopting repo: `lint` in the
  `ci` script, no lint job). Fix whichever is wrong in the same change that
  notices it.

No browser automation belongs in this baseline. A repo missing any gate above
has an unmet L1 obligation, and closing it precedes any stronger verification
tier: a browser suite in a repo without a working CI job gates nothing.
Evidence routing and test-scope judgment are L2 — see `../conventions.md`.

## Generated files

Generated artifacts (route trees, ORM output, lockfiles) go in the
formatter's `ignorePatterns` and never receive hand edits. Regenerate,
don't patch.
