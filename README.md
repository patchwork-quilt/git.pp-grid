# git.pp-grid — The Grid

**One web page where a group coordinates who covers what, when — a custody month, an on-call rotation — from a single shared record that can explain every cell, instead of from scattered messages nobody can reconstruct.**

**Live: https://gitpp-grid.superinstance.dev** (leads with the on-call instance; the custody instance is at `/?instance=custody`)

For: small groups whose schedule is a negotiation — parents sharing a kid's month, a team sharing on-call — where "who has Friday?" needs an answer *and* a reason. Not for: anyone who needs accounts, sign-in, or permissions — there are none here (see the limits, altitude 2). If you want the git-native engine this surface is a sibling of, that's [git.pp-ng](https://github.com/patchwork-quilt/git.pp-ng).

## Try it in thirty seconds

Open the live page. Before any grid, the page states its headline facts in plain text, rendered on the server — right now, on the on-call instance:

```
Freeze: Billing v2 launch freeze — billing, 2026-10-19 to 2026-10-23.
Uncovered: search on 2026-10-31 — nobody stands there.
Pending proposal oc-10: ingest 2026-10-24 → NOBODY (uncovered) — proposed by Priya Nair (tentative)…
```

Or pull one service's next 72 hours as JSON — a machine-readable answer, no page at all:

```
$ curl -s "https://gitpp-grid.superinstance.dev/pull?instance=oncall&seat=pager&row=search&format=json"
{"view":"pager","instance":"oncall","row":"search","window":{"from":"2026-10-12T09:00:00.000Z","to":"2026-10-15T09:00:00.000Z"},
 "entries":[{"date":"2026-10-12","person":"leo","name":"Leo Park",…}, …],
 "digest":"20c86fabce7cdf84d6…"}
```

What just happened: the page and the JSON are two readings of the same thing — a default rotation plus an append-only record of proposals and rulings. Nothing on either is stored as "the schedule"; both are folded from the record on request, so any answer can be traced back to the entries that produced it.

---

## Altitude 2 — the model, in the order it composes

- **Cells.** A keyed space (rows × days here). A cell holds one standing value — a person, or null, a gap — plus annotations.
- **A default rule, as data.** A declared rotation per row, anchored to a date ("weekly rotation, handover Mondays 09:00; billing — Tom → Ana"). The standing grid is *derived* from it, never stored as facts. Silence inherits: if nobody proposes anything, the default is simply what happens.
- **An append-only record.** Authored, reasoned entries; nothing is edited or deleted. A rejection keeps its reason attached forever, because the fold reads the whole log, always.
- **Proposals.** Pending deltas against cells, rendered as a *colour*, not a fact. Five moves: **propose, accept, reject** (a reason is **required**, enforced server-side), **counter, question/comment**. A proposal lands only when *another* seat's ruling enters the record — the code refuses self-rulings (`you cannot rule on your own proposal`, `src/index.ts`). Until then it is inert: visible, explained, and without effect.
- **Freezes.** Declared windows over cells. A frozen cell renders locked, and a proposal against it must **name the freeze in its note** or the port refuses it (`src/index.ts`, message in `src/engine.ts`).
- **Seats.** Server-side projections of the same record: *person* (the grid at their scope), *scoped* (only cells carrying chosen event kinds — the rest is **absent** from the payload, not hidden), *aggregate* (counts and gaps only, no names at all), *agent* (a need-to-know list of dings), *pager* (a machine pull: one row, the next 72 hours, `.ics` or JSON).
- **Ports.** Pull, don't push. The only way in is appending a record through a declared action (`POST /api/grid`: propose / accept / reject / counter / question / comment / optin, plus status / refold / reset); the way out is a pull artefact (`.ics` / JSON) generated from the fold.
- **Cell insides.** Every cell opens: the *description* (what the default says, what stands now, any freeze) and the *equation* (every record that touched the cell, reasons attached, rejected ones struck through).

### The two instances (config in `src/instances.ts`)

**On-call rotation** (the lead page) — services (api, billing, ingest, search) × days, a weekly rotation per service. The seed month is mid-story, and the facts block above is its current headline: a Billing v2 launch freeze **Oct 19–23**; **search on Oct 31 uncovered** (an accepted drop left it open); a swap already rejected *because* "that's the launch freeze"; and proposal **oc-10** pending — Priya asking to leave ingest uncovered on Oct 24. Seats: four engineers scoped to their rotation's services, a team lead (all services + inbox), a **director** (per-service coverage, gaps and pending counts — literally no names in the payload), and the **pager** used in the example above.

**The shared month** (custody, `/?instance=custody`) — two parents, one kid, alternating weeks from Friday Oct 2, soccer standing Tue/Thu. Its facts block today: the current week stands with Jonas except Monday Oct 12 (Mara), and proposal **seed-09 is pending** — Oct 26 currently stands with Jonas; Jonas proposes it go to Mara. Seats: both parents, a **coach** who receives only soccer/snack days (no responsibility field exists in that payload), and an **agent** seat (dings only). This was the original demo; re-expressed as engine config, its playtests are the regression suite.

### Honest limits, early

- **No accounts, no authentication.** A "seat" is a named projection, not a permission: anyone who can reach the page can view any seat and post as any seat. This is a coordination surface for groups that already trust each other, not an access-controlled system.
- **Not the git.pp substrate.** Each instance's record lives in its own Durable Object SQLite — *an instance with the same operational fiction*, not git-backed repos. What is under test here is the shape: defaults as data, proposals as colours, seats as projections. The git-native sibling is [git.pp-ng](https://github.com/patchwork-quilt/git.pp-ng).
- **The `reset` action restores an instance's seed record.** It exists so the demos can be re-run; on a record you cared about, it would be data loss by design. Treat the live instances as exhibits.
- **The facts snapshot is headline-only by design**: the current week, freeze windows, uncovered cells, pending proposals. Everything else is in the grid and the record — the snapshot is a door, not the whole house.

---

## Altitude 3 — the deep layer

**The facts are server-rendered from the same fold the API serves.** The page's plain-text snapshot is generated by `factsSnapshot()` (`src/page.ts`) from `deriveGrid` over the raw log — the identical derivation behind `/api/grid`. There is no second source of truth to drift. A reader who cannot or does not run the client — a text-only agent, a search index, a screen reader — meets the facts the page exists to show, near the top of the HTML (byte ~6,100 of ~34,000, before the grid table; the playtest checks the offset, S6). This mattered in practice: an earlier build of this page presented an empty shell to exactly those readers, and the programme's cold-agent surface tests scored it a failure while every API check passed.

**The page once shipped dead, and no test could see it.** Two of the page's three inline script blocks carried a syntax error (a template-escape slip), so the interactive client never ran — while fetch-based checks, which never parse the scripts, all passed. The fix is now a permanent gate, not a memory: playtest **F7** compiles every script block in the served page with `vm.Script` and fails the run if any block doesn't parse, and the suite additionally executes the first block in a VM and renders a cell with it (`test/playtest.mjs`). If you take one practice from this repo, take that one: *a page is only a page if its scripts parse.*

**A proposal and a dispute are the same shape at two ends of a record's life.** In the Grid, an open proposal is inert — a colour, not a fact — and bites only when a *settling record* (another seat's accept or reject) closes it. The git.pp family's dispute design is that shape noticed from the other end: an open dispute over an existing record, equally inert until a settling record closes it. (Stated as lineage, precisely: disputes live in [git.pp-ng](https://github.com/patchwork-quilt/git.pp-ng)'s design; this repo implements the proposal half, and the word "dispute" appears nowhere in its code. The convergence is the design seed the rest of the family grew from.)

**The derived grid is a cache, and it can prove it.** Discard the derivation and re-fold from default + record: the sha256 digest over every cell's value, freeze and events must come back identical, or something other than the record was carrying the truth (`refold` action; playtest checks it per instance). Relatedly, the realtime socket broadcasts only `{version, digest}` pings, so a connected page re-fetches *its own seat's* projection — a scoped seat's need-to-know is a fact about the payload it receives, never a client-side promise.

### Verify it yourself

```
GRID_URL=https://gitpp-grid.superinstance.dev node test/playtest.mjs
```

The test never lets the server grade itself: it re-folds each instance's raw record from the rule written out in the test, compares every cell, then probes the story — swaps, the freeze guard, director name-freedom by grep, the pager window, scoped seats, the pulls, re-fold digests, the script-parse gate, and the facts snapshot's presence and placement. Latest run, against the live deployment:

**custody 11/11 · on-call 13/13 · face 7/7 · SSR facts 6/6 · total 37/37.**

### Layout

```
src/engine.ts     the engine: fold, projections, pulls, digest (no instance vocabulary)
src/instances.ts  the instances: ON-CALL + CUSTODY configs and their seed records
src/index.ts      the host: one Durable Object (Agents SDK) per instance, the port's guards, HTTP front door
src/page.ts       one generic page, rendered per instance from its config (facts snapshot lives here)
test/playtest.mjs the acceptance test — folds the raw record independently, per instance
```

One Durable Object per instance, the record in its own SQLite, derivation per request.

## Run it

```
bun install
bunx wrangler dev        # local
bunx wrangler deploy     # publish
```

## Siblings — one tool, one repo

- [git.pp-ng](https://github.com/patchwork-quilt/git.pp-ng) — the spec and fold engine: provenance-carrying records on plain git, with the conformance corpus every port must pass. The substrate this surface is the human face of.
- [git.pp-desk](https://github.com/patchwork-quilt/git.pp-desk) — a read-only desk over git.pp projects: state, provenance and closure digests, no write path.
- [gitpp-action](https://github.com/patchwork-quilt/gitpp-action) — the fold inside your own CI, as a GitHub Action: advisory derived state on the records in a repo.
- [gitpp-app](https://github.com/patchwork-quilt/gitpp-app) — the GitHub App: the same derivation, posted as Checks on pull requests.
- [outbound-scanner](https://github.com/patchwork-quilt/outbound-scanner) — a different tool on the same shelf: a pre-send fence that stops an agent narrating a session secret into chat.

MIT © SuperInstance. Part of the git.pp family.
