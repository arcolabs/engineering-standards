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
pass one re-wraps, pass two reaches the fixed point. A single-pass format
followed by CI `format:check` can flap on that pattern. Recheck when
upgrading oxfmt; delete the second run once upstream is idempotent.

## CI gates

Every PR and every push to main runs, as separate jobs: `format:check`,
`typecheck`, `lint`, `test`, `build`. A green merge button requires all of
them. Gates are deterministic and hermetic: no network-dependent checks in
the required path.

## Generated files

Generated artifacts (route trees, ORM output, lockfiles) go in the
formatter's `ignorePatterns` and never receive hand edits. Regenerate,
don't patch.
