# Las Vegas GP 2026 — F1 Race Simulator

> A zero-dependency, single-file F1 race simulator rendered entirely on HTML5 Canvas. No frameworks. No npm install. No excuses.

---

## Features

- **22-car field** with all 11 real 2026 F1 teams and driver codes
- **Full race simulation** over 50 laps of the Las Vegas Strip Circuit (6.201 km)
- **Tyre strategy model** — Soft / Medium / Hard compounds with lap-life degradation and ±15% variance per driver
- **Pit lane state machine** — `racing → entering → stopped → exiting → racing`, with correct arc-length rejoin
- **Live timing sidebar** — gap to leader in fractional laps, pit badges, lap counter, race clock
- **Podium screen** on chequered flag — P1/P2/P3 with team colours, block heights encoding rank
- **Speed controls** — 0.5×, 1×, 2×, 4×, and pause. Pit stop duration is wall-clock fixed so it is not skipped at 4×
- **Neon Vegas aesthetic** — `#05050a` background, CSS grid layout, JetBrains Mono, Bebas Neue, Monoton fonts

---

## How It Works

### Track Geometry: Closed Catmull-Rom Spline

The Las Vegas Strip Circuit is modelled as 18 control points connected by a **closed Catmull-Rom spline** (uniform parameterisation, α ≈ 0.5):

```
p(t) = 0.5 * [ 2P1
             + (-P0 + P2) t
             + (2P0 - 5P1 + 4P2 - P3) t^2
             + (-P0 + 3P1 - 3P2 + P3) t^3 ]
```

Arc length per span is approximated by a **Riemann sum** (N=24 samples). The cumulative arc-length array `cum[]` lets `sampleTrack(s)` do **binary search in O(log n)** to locate the correct span, then linearly interpolate the local parameter `t`.

### Position Tracking

Each car's state is a single scalar `s` — arc-length along the track. Lap count is `floor(s / TOTAL_LEN)`. Gap to leader is `(leader.s - car.s) / TOTAL_LEN` in fractional laps. Position, heading, and render coordinates all derive from a single call to `sampleTrack(s)`.

### Pit Lane State Machine

```
racing   --(needsPit && crosses PIT_ENTRY_S)--> entering
entering --(pitS >= PIT_BOX_S)----------------> stopped   (150 frames @ 60fps = 2.5s)
stopped  --(pitFrames == 0)-------------------> exiting   (new tyres assigned)
exiting  --(pitS >= PIT_LEN)-----------------> racing    (rejoin at PIT_EXIT_S)
```

The pit lane is a separate set of linear segments with its own arc-length parameterisation. On exit, `c.s` is set to `currentLap * TOTAL_LEN + PIT_EXIT_S` to preserve lap count without resetting progress.

### Tyre Degradation

Each car gets a `tyreLife` sampled from `baseLife * Uniform(0.85, 1.15)` on mount. Once `lapsOnTyre >= tyreLife`, `needsPit` is flagged and the car commits to a stop at the next crossing of `PIT_ENTRY_S` (93% of lap). A new compound is randomly assigned on exit.

### Render Pipeline

Per frame:

```
clearRect -> drawTrack() -> tickCars() -> drawCars() -> [drawPodium()] -> renderBoard()
```

`renderBoard()` runs every 6th frame to avoid DOM thrash. Track path is rebuilt each frame as a `Path2D` with 800 sample points. Car render complexity is O(n).

---

## Usage

```bash
# Option A: open index.html directly in any browser

# Option B: serve locally
python3 -m http.server 8080
# http://localhost:8080
```

No build step. No `package.json`. The only external requests are Google Fonts.

---

## Controls

| Control | Action |
|---|---|
| `0.5x` / `1x` / `2x` / `4x` | Simulation speed multiplier |
| `PAUSE` / `RESUME` | Freeze/unfreeze the race |

---

## 2026 Driver Grid

| Team | Driver 1 | Driver 2 |
|---|---|---|
| Mercedes | ANT (Antonelli) | RUS (Russell) |
| Williams | ALB (Albon) | SAI (Sainz) |
| Haas | BEA (Bearman) | OCO (Ocon) |
| Ferrari | HAM (Hamilton) | LEC (Leclerc) |
| Red Bull | VER (Verstappen) | HAD (Hadjar) |
| Racing Bulls | LAW (Lawson) | LIN (Lindblom) |
| Aston Martin | ALO (Alonso) | STR (Stroll) |
| Alpine | GAS (Gasly) | COL (Colapinto) |
| Cadillac | BOT (Bottas) | PER (Perez) |
| McLaren | NOR (Norris) | PIA (Piastri) |
| Audi | HUL (Hulkenberg) | BOR (Bortoleto) |

---

## Tech Stack

| Layer | Choice |
|---|---|
| Renderer | HTML5 Canvas 2D API |
| Track geometry | Catmull-Rom spline + arc-length reparameterisation |
| Position model | 1D arc-length scalar per car |
| Pit logic | 4-state FSM |
| Fonts | JetBrains Mono, Bebas Neue, Monoton (Google Fonts) |
| Dependencies | **0** |

---

## License

MIT
