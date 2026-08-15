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
| Training | `training` | Day cards + log modal |
| Nutrition | `nutrition` | Macro tracker + food chips |
| Weight | `bodyweight` | Body weight log |
| Log | `logview` | Calendar + day detail |
| Progress | `progress` | Per-exercise weight/rep graphs |
| Info | `principles` | Key principles text |

Tabs in `<div class="tabs-wrap">`. `showTab(id, el)` activates a section.

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

### weightLogs: `[{date, kg}]`
### nutLogs: `{[dateStr]: [{name, kcal, prot, carb, fat}]}`

## DAYS data (source of truth for exercises)
```
A: Day A — Lower Body, Calves & Core
  A-climb  Bouldering / Climbing       session (not weighted)
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
  B-ohp    Barbell OHP                 4 × 6–8
  B-inc    DB Incline Bench            3 × 8–10
  B-dips   Weighted Dips               3 × 8–10
  B-lat    Lateral Raise               3 × 12–15
  B-dd     Dead Bug                    2 × 10/side (bw)
  B-calf   Seated Calf Raise           4 × 12–15

C: Day C — Posterior Chain & Arms
  C-climb  Bouldering / Climbing       session (bw)
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

## Key JS functions
| Function | Purpose |
|---|---|
| `showTab(id, el)` | Switches active section |
| `openLogModal(day)` | Opens workout log modal for day A/B/C |
| `saveSession()` | Saves current logState to workoutLogs |
| `renderProgressTab()` | Renders exercise list in Progress tab |
| `openProgressChart(exId, exName)` | Shows weight/rep graph for exercise |
| `renderNutTracker()` | Re-renders nutrition tracker for today |
| `renderCalendar()` | Re-renders log tab calendar |
| `exportData()` / `importData()` | JSON backup/restore |
| `save(k, v)` / `load(k, d)` | localStorage helpers (prefix `cl4_`) |

## Export/import format
Version 4. Import merges by date+day key (existing entries not overwritten by import unless same date+day). New fields on exercise objects are additive — old exports without them still import safely. Never bump `_version` for additive-only changes.

## Common pitfalls
- **Branch**: harness resets to stale feature branch — always `git checkout main` at session start before reading or editing files.
- **iOS double-tap zoom**: `touch-action: manipulation` on html/body + `maximum-scale=1` in viewport meta.
- **iOS input zoom**: `font-size: 16px` on all inputs (below 16px triggers auto-zoom on iOS Safari).
- **iOS :hover persistence**: use `:active` not `:hover` for delete/action buttons to avoid sticky red X bugs on touch devices.
- **cherry-pick caution**: don't cherry-pick commits from the stale feature branch onto main without careful inspection — the stale branch has old/diverged code.
