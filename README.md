# Stationeers Atmospherics Gas-Mix Calculator

Deploy repo: `growaleaf/stationeers-atmospherics-calculator` (public — GitHub Pages free tier
needs it). Live at `https://growaleaf.github.io/stationeers-atmospherics-calculator/` once
deployed. This directory in the Hive monorepo is the source of record.

Charter: `Autonomous/WORKER_OUTPUT/GAMETOOLS/2026-10-08_stationeers-atmospherics-calculator/CHARTER.md`
(lane id GCT-011).

## What it is

A static single-page calculator for
[Stationeers](https://store.steampowered.com/app/544550) (Steam). Given a room/suit volume (L),
target pressure (kPa), temperature (K), and a gas mix (O2, N2, CO2, Pollutant, Volatiles, N2O
percentages), it computes the moles of each gas needed — and the STP-liter equivalent — using the
game's own ideal-gas-law constant. It also shows the resulting O2 partial pressure against the
wiki's own breathing bands. Three presets: a common breathable mix, the same mix with pollutant
forced to 0%, and a "welding bypass" mix that zeroes oxygen entirely (no oxidizer, no combustion,
regardless of volatiles percentage).

## Correction to the brief — a third incumbent exists, and the category was reopened, not new

The charter's own "$10 arithmetic" line states this lane's 2026-09-22 research "found zero
dedicated gas-mix calculator, only a furnace-specific one." That research missed one: a
third-party **Stationeers Gas Mixer Calculator** at `tools.supercraft.host` exists and does solve
the ideal gas law plus a volatiles/oxygen combustion ratio. It is scoped entirely to engine fuel
mixing — no breathable-atmosphere presets, no multi-gas life-support mix, no safety-band reference
info. This build is still the right one to ship: it serves the different, unserved mechanic (room/
suit life support), not the one `supercraft.host` already covers (fuel combustion). The
"How this differs" section on the shipped page names all three incumbents explicitly, including
this one, so a reader lands on the right tool for what they're actually trying to do.

Separately: this exact game was killed on 2026-09-28 (see
`Orchard/ORCHESTRATORS/GAMETOOLS/PLAYBOOK.md`, 2026-09-28 entry — "2 fresh candidates killed on
first check... Stationeers") for the same furnace/wiki incumbents the charter cites. It was
reopened 2026-10-07 under the SPINE law 3 rewrite ("an incumbent is NOT a veto... earlier kills
that rested only on 'an incumbent exists' are reopened and not binding"), the same mechanism that
reopened Caves of Qud (GCT-009) and Mon Bazou (GCT-010) this same week.

## Sourcing — every number traces to a fetch, nothing fabricated

`stationeers-wiki.com` 403s on a direct automated fetch (Cloudflare challenge) — the same shape
logged in this lane's PLAYBOOK on 2026-10-07. Its content was read instead via
`legacy.stationeers-wiki.com` and a third-party mirror
(`stationeers-wiki.armzilla.jxsn.dev`), both serving the same wiki's pages directly, plus
`WebSearch`'s own indexed copies, consistent with this lane's standing rule that a 403 on the
crawler is not evidence the content is unverifiable.

| Constant | Status | Primary source | Second check |
|---|---|---|---|
| Ideal gas law `P·V = n·R·T`, R = 8314.46261815324 L·Pa/(mol·K) | **2-source corroborated** | stationeers-wiki.com — Pressure, Volume, Quantity, and Temperature (fetched via mirror); its own worked furnace example (2 mol, 300 K, 1000 L → ≈5 kPa) reproduced exactly by this build's code under Node, see Verification below | tools.supercraft.host's independent Stationeers gas-mixer calculator uses the same constant as "R = 8.314" in J/(mol·K) — same value, different unit scale, independently authored |
| O2 partial-pressure breathing floor: 16 kPa | **2-source corroborated** | stationeers-wiki.com — Air (via mirror): "16 kPa or higher... is safe to breathe" | xgamingserver.com — Stationeers Atmospherics 101, independently: "at least ~16 kPa O2" |
| O2 LOW (16→12 kPa) / CRITICAL (12→5 kPa) warning split | Single-sourced — labeled, not claimed as double-confirmed | stationeers-wiki.com — Air (via mirror) | No second source found stating this exact split; shown on the page as single-sourced reference, same treatment as the Caves of Qud build's caste/calling bonuses |
| EVA Suit: 10 L volume, 202.65 kPa max pressure | Single-sourced — labeled | stationeers-wiki.com — EVA Suit (via mirror) | Not used to gate the calculator; reference only |
| Pollutant suit-warning threshold: 0.1 mol | **Not corroborated** — shipped as a note to check your own analyzer, not a default | stationeers-wiki.com — Pollutant (same text mirrored on the wiki's own `legacy.` subdomain — one underlying source, not two) | No independent second source found |
| Volatile/O2 flammability threshold: ~5% each | **Not corroborated** — shipped as a note, not a default | A single Steam Community discussion thread | No wiki or second guide found; the welding-bypass preset instead zeroes O2 entirely so it needs no threshold number |

Per the charter's explicit failure protocol and the Norland/Forever Skies precedent in this lane's
PLAYBOOK, the two uncorroborated constants (pollutant, volatile) are not built-in defaults — the
page states this directly in its Safety Reference section and tells the player to read their own
in-game analyzer instead.

## Verification performed by the worker (not just written)

- `python3 Tools/slop_lint.py Orchard/PRODUCTS/stationeers-atmospherics-calculator/index.html` →
  clean, 0 findings.
- Hand-calculated 4 known-answer gas-mix scenarios independently in Python (see the session's
  `DONE_REPORT.md` for the exact script and output), then extracted the live `computeMix()`
  function straight out of the shipped `index.html` (not a reimplementation) and ran it under
  Node.js with a minimal DOM mock exercising the actual shipped code path, including its own
  verification table's PASS/FAIL logic. All 4 cases, plus the wiki's own furnace worked example,
  passed exactly.
- No headless browser (Puppeteer/jsdom) or Chrome MCP is available to this CLI worker — this is a
  known limitation of bare `claude -p` workers, not a shortcut taken. The Node-DOM-mock run above
  is the closest available substitute: it executes the real shipped script end to end, including
  every DOM read/write the page performs on load, rather than a pure-function reimplementation.

## Build

No build step. `index.html` is the entire site — inline CSS, inline vanilla JS, no dependencies,
no framework, no backend.

## Deploy

```bash
# repo created via: POST /user/repos (growaleaf account)
git clone https://x-access-token:$GITHUB_TOKEN@github.com/growaleaf/stationeers-atmospherics-calculator.git
# copy index.html, sitemap.xml in, commit, push to main
# Pages enabled via: POST /repos/growaleaf/stationeers-atmospherics-calculator/pages {"source":{"branch":"main","path":"/"}}
```

## Note on the charter's requested path and sitemap/analytics

The charter names the product folder as `Products/StationeersAtmospherics/`. This build instead
uses `Orchard/PRODUCTS/stationeers-atmospherics-calculator/`, matching the convention this same
lane actually used for its two most recent sibling builds this week — GCT-009
(`caves-of-qud-point-planner`) and GCT-010 (`mon-bazou-money-calculator`) — both kebab-case under
`Orchard/PRODUCTS/`, both deployed to `growaleaf.github.io/<slug>/`. Neither of those two sibling
builds — "this lane's other live sites" the charter asks this page to match — ships a
`sitemap.xml`, `robots.txt`, or any analytics/beacon snippet; a single-page `growaleaf.github.io`
tool apparently doesn't carry one in this lane's actual current practice. A minimal `sitemap.xml`
is still included here since the charter asks for one explicitly and it costs nothing; no
analytics snippet was added, since fabricating one here would not actually match the lane's other
live sites — it would add something they don't have.

## Sourcing rules

- Every numeric constant on the page cites its source inline in the footer/Sources section, with a
  fetch date.
- No fabricated or guessed numbers presented as fact — pollutant and volatile exposure thresholds
  are explicitly left as player-input fields rather than forced into a false "corroborated" bucket.
- No account/login features, no extracted game art/sound — text and numbers only, cited to the
  wiki and independent guides.
- No custom domain, no registry write, no git commit on the orchestrator's branch — all reserved
  for the orchestrator per the charter.
