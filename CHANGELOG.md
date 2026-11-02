# Changelog

All notable changes to git.pp-grid are recorded here. The version in
`package.json` is the version of record; release tags mirror it.

## [0.1.0] — 2026-11-02

First tagged release: the Grid as it has run live since October 2026.

### The engine

- **2026-10-14 — the Grid itself** (`1e238cc`): a default rule plus an
  append-only record, folded on request into a derived month; per-seat
  renderings (full grid, server-side events-only projection, agent dings
  list); proposals rendered as pending colour with accept / reject
  (reason enforced) / counter / question; pull-my-days `.ics`; a re-fold
  digest proof; a seeded October 2026 story (accepted swap, rejected
  Halloween with reason, pending tentative + question). Playtest 11/11
  live, including an independent fold at 31/31.
- **2026-10-14 — generalised** (`3abdec8`): one engine, many instances.
  Instances are config — the on-call rotation (lead) and the shared
  month (custody) run on the same code.

### Face and facts

- **2026-10-15 — face-grade fixes** (`4b66e2e`): latest settlement reason
  and open-proposal note on the cell face; counter-proposal affordance
  in action rows and the port.
- **2026-10-15 — SSR facts snapshot** (`8cd1e55`): headline facts
  server-rendered as plain text in the page, from the same fold the API
  serves.
- **2026-10-15 — facts placement** (`755c362`): the snapshot sits
  immediately after the header/intro, before the explainer cards and
  the grid; the playtest's S6 guard checks the byte ordering.

### Documentation and CI

- **2026-10-22 — README rewritten** (`c40dd74`) to the three-altitude
  standard, against live-verified facts and a 37/37 playtest receipt.
- **2026-11-02 — CI added** (`.github/workflows/ci.yml`): install +
  typecheck on push and pull request. The acceptance playtest stays
  live-only and manual — it grades a deployment, re-folding the record
  from the server's responses, so CI does not pretend to run it:
  `GRID_URL=https://gitpp-grid.superinstance.dev node test/playtest.mjs`.

### Branch state at this release

The four topic branches — `face-reasons-counter`, `grid-ssr-snapshot`,
`facts-placement`, `readme-standard` — were merged into `main` through
PRs #1–#4 and are part of this release. Verified at release time
(`git branch -r --no-merged main` is empty): no unmerged feature
branches exist, so nothing is parked or held back from 0.1.0.
