# Climb & Lift — Context for Claude

## What this is
Single-file vanilla JS/HTML/CSS PWA at `index.html`. No build step, no dependencies.
Deployed on GitHub Pages at `matikalter.github.io/climb-and-lift` from the `main` branch.
All data stored in `localStorage` with key prefix `cl4_` (via `save(k,v)` / `load(k,d)` helpers).

## Git workflow
**Always develop on `main`.** The harness auto-checks-out branch `claude/add-food-options-quantities-uUkBE` at session start — run `git checkout main` before making any edits. That branch is stale and should be ignored.

## App structure
| Section | id | Notes |
|---|---|---|
| Training | `training` | Day cards + log modal + today's logged sessions inline |
| Nutrition | `nutrition` | Macro tracker + food chips, **then body weight log**, then meal plan |
| Log | `logview` | Calendar + day detail |
| Progress | `progress` | Per-exercise weight/rep graphs |
| Info | `principles` | Key principles text |

Tabs in `<div class="tabs-wrap">`. `showTab(id, el)` activates a section.
There is no Weight tab — the body-weight logger lives inside the Nutrition section
(`#bwInput` / `#bwSvg` / `#bwList`), and `showTab('nutrition')` calls `renderBwList()`.

## Design system
```
--bg: #0f0f0d         dark page background
--surface: #1a1a18    card background
--surface2: #2a2a28   input / chip background
--border: (implied)   muted borders
--accent: #d4df64     lime green — primary highlight
--accent2: #d4b060    gold — secondary / star / markers
--danger: #ff6060     red — delete / over-limit
--muted: muted grey text
--text: main text
--green: success bar fill
```
Font: `DM Sans` (body), `DM Mono` (labels/badges), `Bebas Neue` (headings).
iOS web app: `viewport-fit=cover`, `env(safe-area-inset-top)`, `touch-action: manipulation` on body to disable double-tap zoom. Inputs use `font-size: 16px` to prevent iOS auto-zoom.

## Data structures

### workoutLogs: array of session objects
```json
{
  "date": "2026-01-15",
  "day": "A",
  "dayName": "Day A — Lower Body, Calves & Core",
  "exercises": [
    {
      "id": "A-squat",
      "name": "Barbell Back Squat",
      "sets": "4 × 6–8",
      "done": true,
      "weight": "80",
      "reps": "7",
      "hitTarget": true,
      "w": true
    }
  ],
  "notes": "",
  "_version": 4
}
```
`hitTarget` (bool) = user marked they hit the target rep range at that weight. Additive field — old logs without it default to false.

`reps` is **free text**, not a number — the input is `type="text"` so a per-set
breakdown like `"6-6-3-3"` can be entered and is stored and displayed verbatim.
Nothing does arithmetic on it. Because it reaches markup in several places, run it
through `escHtml()` / `escAttr()` when interpolating (the badge, the tooltip, and
the input's `value=`).

### weightLogs: `[{date, kg}]`
### nutLogs: `{[dateStr]: [{name, kcal, prot, carb, fat}]}`

## DAYS data (source of truth for exercises)
```
A: Day A — Lower Body, Calves & Core
  A-climb  Bouldering / Climbing       session (not weighted)
  A-fb     Finger Block — Half Crimp   3 ramps → work sets  (protocol: FB_PROTOCOL, repLabel: sets)
  A-squat  Barbell Back Squat          4 × 6–8
  A-rdl    Romanian Deadlift           3 × 8–10
  A-ht     Barbell Hip Thrust          3 × 10–12
  A-calf   Standing Calf Raise         4 × 10–12
  A-abwheel Ab Wheel Rollout           2 × 8–12 (bw)
  A-pallof  Pallof Press               2 × 10/side (bw)
  A-splank  Side Plank                 2 × 40s/side (bw)
  A-rh     Reverse Hyperextension      3 × 10 (bw)

B: Day B — Push & Shoulders
  B-climb  Bouldering / Climbing       session (bw)
  B-fb     Finger Block — Half Crimp   3 ramps → work sets  (protocol: FB_PROTOCOL, repLabel: sets)
  B-ohp    Barbell OHP                 4 × 6–8
  B-inc    DB Incline Bench            3 × 8–10
  B-dips   Weighted Dips               3 × 8–10
  B-lat    Lateral Raise               3 × 12–15
  B-wcurl  DB Wrist Curl               3 × 12–15
  B-wext   DB Wrist Extension          3 × 15–20
  B-dd     Dead Bug                    2 × 10/side (bw)

C: Day C — Posterior Chain & Arms
  C-climb  Bouldering / Climbing       session (bw)
  C-fb     Finger Block — Half Crimp   3 ramps → work sets  (protocol: FB_PROTOCOL, repLabel: sets)
  C-pu     Weighted Pull-ups           4 × 5–6
  C-dl     Barbell Deadlift            4 × 4–6
  C-row    Single-Arm DB Row           3 × 10–12/side
  C-fp     Cable Face Pull             3 × 15
  C-curl   Incline DB Curl             3 × 10–12
```

## Workout plan rationale
**Goal**: muscle mass + climbing performance + healthy core to eliminate back/hip issues.
- Gym always AFTER climbing (skill work first, fresh fingers)
- OHP prioritised (addresses pull-dominant climbing imbalance)
- RDL (Day A) + Deadlift (Day C): different stimuli — RDL = hamstring hypertrophy under eccentric tension; deadlift = raw posterior chain strength
- Hip Thrust in Day A (with squats/RDL) — correct lower-body context
- Pull-ups FIRST on Day C (before deadlift), both tax grip/lats
- Core: ab wheel (anti-extension), pallof press (anti-rotation), side plank (anti-lateral), reverse hyper (direct erector/multifidus strengthening)
- Dead Bug (Day B): anti-flexion core, harder than Bird Dog, appropriate alongside push work
- Face pulls on Day C: rotator cuff / rear delt, critical for shoulder health in climbers
- Single-Arm DB Row on Day C: horizontal pull, addresses mid-trap/rhomboid gap left by climbing's vertical-pull dominance
- Finger block on **all three days**, always immediately after climbing and **before the lifts** — max-effort finger work needs a fresh grip, and pull-ups/deadlift/row would pre-fatigue it. Target ≥2×/week; it's the finger stimulus on non-climbing days.
- Wrist curl + extension on **Day B only**, not Day C. Day B is the least grip-taxing day, so the forearms are fresh; Day C already stacks pull-ups, deadlift, rows and curls. Wrist extension is the antagonist climbing never trains — the main defence against climber's elbow (lateral epicondylitis).

### Finger block protocol (`FB_PROTOCOL`)
Ramp to a session max: 8 reps @ 50% → 5 @ 65% → 5 @ 80% → 5 @ 100%.
Advanced (optional): 3–10 work sets of 3–8 reps.
Progression: add a work set each week first; once at 5+ work sets, +~1 kg/week.
Logged values are the session max weight and the number of **work sets** (not reps).

`protocol` is an optional field on a DAYS exercise. `saveSession()` only copies
`id/name/sets/done/weight/reps/w/hitTarget` into logs, so `protocol` never reaches
`localStorage` or an export — it can be edited freely without touching the data format.
In the log modal it renders via `protoHtml(ex)` behind a `▾ PROTOCOL` disclosure
(`toggleProto`); `.ex-log-row` is `flex-wrap:wrap` so the panel can take a full-width row.

### `repLabel` — relabelling the second input
Logs always store the second value in the `reps` field; `repLabel` only changes the
word shown for it (the finger block counts **work sets**). `repLabel` lives in DAYS,
not in logs, so `repUnit(id)` / `exDef(id)` look it up by exercise id — meaning old
logs pick up the right word too. `repsBadge(ex)` spells the unit out only when it
isn't the default, so ordinary lifts still read `80 kg × 7` while the finger block
reads `32 kg × 6 sets`. Adding another such exercise is just `repLabel:'...'` — no
data migration, no `_version` bump.

## Key JS functions
| Function | Purpose |
|---|---|
| `showTab(id, el)` | Switches active section |
| `openLogModal(day)` | Opens workout log modal for day A/B/C |
| `saveSession()` | Saves current logState to workoutLogs |
| `renderProgressTab()` | Renders exercise list in Progress tab, grouped by Day A/B/C |
| `openProgressChart(exId, exName)` | Opens the chart view for one exercise |
| `gotoProgress(exId)` | Training tab 📈 button — jumps straight to that exercise's chart |
| `exDef(id)` / `repUnit(id)` | Look an exercise up in DAYS by id |
| `escHtml(v)` / `escAttr(v)` | Escape free-text values (reps) headed for markup |
| `setChartFilter(btn, tf)` | Applies a `1M` / `3M` / `1Y` / `ALL` timeframe |
| `chartDomain()` | Returns `[t0, t1]` — the x-axis time window for the active filter |
| `drawChart()` | Measures the container and rebuilds the SVG to fit |
| `buildProgressSvg(sessions, W, H, t0, t1)` | Builds the SVG itself |
| `buildTimeTicks(t0, t1, plotW)` | Date ticks sized to the span, thinned to fit the width |
| `renderNutTracker()` | Re-renders nutrition tracker for today |
| `renderBwList()` | Re-renders body-weight list + sparkline (inside Nutrition) |
| `renderCalendar()` | Re-renders log tab calendar |
| `exportData()` / `importData()` | JSON backup/restore |
| `save(k, v)` / `load(k, d)` | localStorage helpers (prefix `cl4_`) |

## Progress chart
The chart **never scrolls** — it always redraws to fit the container exactly, in
both portrait and landscape. Getting that right depends on a few things:

- **x is positioned by date, not by index.** `xOf(i)` maps a session's real date
  onto `[t0, t1]` from `chartDomain()`. A bounded filter spans `today - N days →
  today`; `ALL` spans first session → last session. This is what makes a wider
  timeframe condense the points and a narrower one spread them out.
- **Width** comes from `chartWrap.getBoundingClientRect().width`; **height** is
  220px portrait, or ~48% of viewport height (clamped 150–210) in landscape, so
  the chart plus tooltip stay above the fold on a short screen.
- **Density-aware marks**: `minGap` (smallest pixel gap between points) drives dot
  radius and line width, and dots are dropped entirely below 5px so long
  timeframes read as a trend line rather than a chain of beads.
- `window.resize` / `orientationchange` re-run `drawChart()` (debounced 150ms).
- Filter pills carry `data-tf`; ones with no sessions in range get `.empty` (dimmed).
- `progressChartState = {curIdx, allSessions, sessions, filter, exId}` — `allSessions`
  is the unfiltered set, `sessions` is what's currently drawn.
- `progressListScrollY` remembers where the exercise list was scrolled when the
  chart opened; `closeProgressChart()` restores it inside a `requestAnimationFrame`
  (the list has to be laid out again before the scroll target exists). Entering via
  `gotoProgress()` resets it to 0, so the back arrow lands at the top of the list.

## Export/import format
Version 4. Import merges by date+day key (existing entries not overwritten by import unless same date+day). New fields on exercise objects are additive — old exports without them still import safely. Never bump `_version` for additive-only changes.

## Common pitfalls
- **Branch**: harness resets to stale feature branch — always `git checkout main` at session start before reading or editing files.
- **iOS double-tap zoom**: `touch-action: manipulation` on html/body + `maximum-scale=1` in viewport meta.
- **iOS input zoom**: `font-size: 16px` on all inputs (below 16px triggers auto-zoom on iOS Safari).
- **iOS :hover persistence**: use `:active` not `:hover` for delete/action buttons to avoid sticky red X bugs on touch devices.
- **cherry-pick caution**: don't cherry-pick commits from the stale feature branch onto main without careful inspection — the stale branch has old/diverged code.
- **Date parsing**: dates are `YYYY-MM-DD` strings. Parse with `new Date(d+'T00:00:00')` (the `tOf` helper) so they land on local midnight — bare `new Date('YYYY-MM-DD')` parses as UTC and shifts the day in negative-offset timezones. Same reason `fmtYMD` formats by hand instead of using `toISOString()`.
- **Chart overflow**: don't reintroduce `overflow-x:auto` on `.chart-wrap`. The chart is sized to fit; if something overflows, the sizing is wrong.
- **Exercises live in two places**: `DAYS` drives the log modal and Progress tab, but the Training tab day cards are hand-written HTML (`.day-card` → `.exercise` blocks with `ex-name` / `ex-note` / `ex-sets`). Adding or renaming an exercise means editing both, in the same order. The day cards deliberately omit the climbing entry.
- **Day card order**: inside `.day-body` it's focus tags → `+ LOG THIS SESSION` → `#todaySession-X` → phase labels and exercises. The log button is at the top on purpose so it's reachable without scrolling past the whole plan.
- **Graph buttons**: every weighted (`w:true`) day-card exercise carries a `.graph-btn` calling `gotoProgress('<id>')`, sitting right after the `.timer-btn`. Bodyweight exercises get none — the Progress tab only lists weighted lifts. Adding a weighted exercise means adding its graph button too, and the hardcoded id in the card must match the `DAYS` id exactly (nothing validates this at runtime).
- **`.ex-sets` width**: capped at `112px` and allowed to wrap. Every numeric set string fits on one line under that cap (`3 × 10–12/side` is the longest at ~110px); only phrase-style values like `3 ramps → work sets` wrap, which keeps them from crushing the note column. Don't put `white-space:nowrap` back.

## Verifying UI changes
Chromium + Playwright are available in the web sandbox and are the fastest way to
check layout across orientations:
```bash
npx http-server -p 8231 -s &          # file:// blocks localStorage, so serve it
# playwright at /opt/node22/lib/node_modules/playwright
# chromium at /opt/pw-browsers/chromium-1194/chrome-linux/chrome
```
Seed state with `page.addInitScript` writing `cl4_logs` before `page.goto`, then
assert on geometry (e.g. svg width vs container width, `scrollWidth - clientWidth`)
rather than eyeballing screenshots alone.
