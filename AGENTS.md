# AGENTS.md — /Users/srg/test

Scratch directory containing interactive educational physiology simulations.
Single-file HTML apps: vanilla JS + canvas, zero external dependencies, opened
directly in a browser via `open <file>`.

Git repo (branch `develop`). Do NOT commit unless the user explicitly asks.

## Contents

| File | Topic |
|---|---|
| `cardio_wave.html` | Pulse wave in an artery, flat 2D (legacy, kept for comparison) |
| `cardio_wave_3d.html` | Same topic, pseudo-3D: hand-written software renderer (tube mesh 130x20, painter's algorithm, diffuse light, orbit camera with drag/wheel), blood-flow particles, wave decomposition into incident/reflected, mmHg readouts |
| `heart_pump.html` | Heart as two-stage pump: 4 chambers, valves, conduction system with AV delay, gated blood particles along full circuit, Wiggers-style curves (ECG, LV/aorta/atrium pressures, LV volume) |
| `muscle_pump.html` | Skeletal muscle: calf pump squeezing deep vein with valves (modes rest/walk/run) + sarcomere with sliding filaments and temporal summation -> tetanus |

## Hard UI rule

Everything MUST fit on ONE screen with NO scrolling. The user rejected scrollable
layouts explicitly. Pattern used everywhere:

- `body { height:100vh; overflow:hidden }`, flex column, gap 8px
- controls row and status row: fixed compact height
- main scene container: `flex:1 1 auto; min-height:Xpx`
- chart strip below: `flex:0 0 clamp(...)`
- canvases fill containers absolutely; size tracked with `ResizeObserver`,
  context scaled by devicePixelRatio

Never reintroduce page scroll or fixed-height canvases.

## Shared physics & calibration constants

If editing, keep these consistent ACROSS files:

- Wave sim: N=520 nodes, L=1.0 m, dx=L/(N-1), damping exp(-0.9*dt), CFL r<=0.92,
  adaptive sub-stepping each frame. Reflection at right edge = Neumann copy.
  The "incident-only" twin simulation (for wave decomposition) uses a
  first-order absorbing boundary `(1-r)*p[N-1] + r*p[N-2]` plus a damping
  sponge over the last 12% of nodes.
- PWV slider maps 4.5..13.5 m/s (qualitative Moens-Korteweg).
- mmHg calibration: `pressure_mmHg = 93 + p_sim * 24` (MAP 93 mmHg).
- Windkessel particle flow: `v = v_mean + v_puls * exp(-x/lambda)`,
  `lambda ∝ (PWV/6.5)^2` — soft wall damps pulsations fast, stiff transmits them.
- Cardiac cycle fractions of period T (used in heart_pump):
  SA fire 0, atrial systole .03-.16, AV delay until .155, isovolumic contraction
  .155-.215, ejection .215-.475, S2/.475, isovolumic relaxation to .56,
  filling .56-1.
- Muscle: Ca transient decay tau=0.24 s; twitch fusion starts ~5 Hz, fused
  tetanus >=~14 Hz; pump squeeze duty ~42% of step period.

## Conventions

- UI text: Russian. Communicate with the user AND reason out loud in Russian
  (explicit user request).
- No comments in code.
- Dark palette: bg #0a0e1a / canvas #0a0f22 / #0d1226, panels #111731,
  borders #223, muted text #7484ab/#9fb0d8.
  Accents: direct wave #ff6b57, reflected #ffb14d, probe 1 #ff8bd2,
  probe 2 #58c4ff, success #59d98c, warning/danger #ff7b72, gold #ffd166.
- Pedagogical patterns the user responds well to — reuse them:
  - numbered live captions explaining current phase (①②③...)
  - cycle-phase timeline bar (systole/diastole with event markers + playhead)
  - preset buttons (e.g. young aorta / age 50 / sclerosis)
  - slow motion default x8-x10
  - decomposition overlays (total vs component curves) with highlighted
    difference zones
  - live numeric readouts tied to chart reference lines

## Environment gotchas (macOS host)

- `node` is NOT installed. To syntax-check an inline `<script>` before
  delivering:
  1. extract script body with python3 regex to a temp .js file
  2. run `osascript -l JavaScript tmp.js`
  3. `ReferenceError: Can't find variable: document` means syntax OK
     (parse succeeded; DOM absent outside browser). Any SyntaxError is real.
- JXA's older JavaScriptCore rejects the `**` exponentiation operator even
  though browsers accept it. Use `Math.pow` in delivered code so this checker
  stays reliable.
- After writing/updating a file, open it with `open <file>` so the user sees
  the result immediately.
