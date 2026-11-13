# TODO — project work before / around the conference

Action items only. Theory: [`CONFERENCE_THEORY.md`](CONFERENCE_THEORY.md).
Items: `W0`, `W-N1`, … — work through them in order (see [Order](#order)).

---

## Open

### W-N2 / W-N3 / W-N4 — optional leftovers

- [ ] W-N2: explicit trajectory fields in `asteroid_state` — not now.
- [ ] W-N3: run the grid from the server API — not now.
- [ ] W-N4: Sobol indices — only if time remains.

### W-P1. Energy-dependent cross sections — ~1 week

**Not “add a second k”** — photon and GCR are already split. Missing: separate k / cross sections for protons, α, HZE and sensible UV (not bolometry). → research **R5**

GCR 90/9/1 % split today is report-only — all components share one k.

### W-P2. Spallation / secondaries — 1–2 weeks

Shielding is monotonic `exp(−kρx)`. Without a secondary cascade the model cannot find a non-trivial thickness optimum from radiation alone (only erosion vs attenuation). → research **R3**. High cost — likely after the conference.

### W-P3. Ice sublimation — 3–5 days

`ICE_RICH` has water; radius changes only via dust erosion. Add mass loss from sublimation in the same `dR/dt` hook as erosion. → research **R4**

### W-P4. Broader biological database — several days

W1b first (done). Then newer data → research **R6**.

### W2. Demo / presentation (in parallel with W-N*)

- [ ] `npm test` (web) + `model/tests/` green.
- [ ] `python tools/export_simulation_to_web.py --check`.
- [x] Ready **long offline replay** (escape / longer timescale) — `web/public/data/run_100kyr.json` and `run_1myr.json`, selectable under Analysis.
- [ ] Confirm offline operation (Gaia / Horizons cache).
- [ ] Optional: rename or compute UV fraction (`uv_*` vs bolometry).
- [ ] Optional: align meaning of `cumulative_exposure` (today excludes GCR).
- [x] `radiation_surv_coeff` slider in UI (most honest knob) — the `c_rad` control in the analysis dock and presentation chapter 5.

---

## Research — what still needs to be found

Only items without which a given task cannot be closed honestly. Everything else is engineering / team decision.

| ID | What to find | Why | Blocks |
|---|---|---|---|
| **R3** | Simplified, citable dose-vs-thickness model with **buildup / spallation** (not full MCNP) — e.g. empirical “dose at center vs g/cm²” curve for GCR at our scale | Without this W-P2 is guessing the curve shape | W-P2 |
| **R4** | Ice sublimation rate in vacuum (Hertz–Knudsen or equivalent) + saturated vapor pressure vs T, with citation | Wire `dR/dt` for `ice_rich` | W-P3 |
| **R5** *(only if doing W-P1 / real UV)* | Tabulated attenuation / stopping power: UV vs protons vs α vs HZE at our scale **or** UV fraction in the stellar spectrum (not bolometry) | Avoid inventing separate `k` values from thin air | W-P1, optional UV in W2 |
| **R6** *(only if doing W-P4)* | Radio-resistance / DNA-repair data newer than Mileikowsky 2000 for extremophiles | Broader biological basis | W-P4 |

**No research needed for:** W2 (except optional UV → R5).

---

## Done

- **Already in code before this list:** escape / collision / Hill-sphere arrival detection (`scenarios.py`); “single k for GCR and photons” fix; “gamma without citations” fix.
- **W0 — cleanup:** radius range `radius_min` / `radius_max` (`server.py`, `controlPanel.js`, `rangeLog.js`; `TestServerRadiusWiring` / `TestFragmentRadiusBounds` green, 2026-08-27); stray root `package-lock.json` removed; README updated (gamma + GCR resolved with residual); “no Python tests” sentence removed from `TECHNICAL_DOCUMENTATION.md`.
- **W1a / R1 — hydrolysis:** `Ea = 130 kJ/mol`, `A = 2.3e11` 1/s (Lindahl & Nyberg 1972).
- **W1b — `radiation_surv_coeff`:** chosen at the time as `3.6e-4 … 1.0e-3` 1/Gy (Mileikowsky D10 → natural-exp, option C). The code now samples `2.5e-5 … 4.3e-4` 1/Gy — see `biology/constants.py`.
- **W1c / R2 — constant 1200:** no source found → kept as **AUDIT / sensitivity** parameter in `biology/constants.py`.
- **W-N1 — ensembles:** seed loop + aggregation (`ensembles/runner.py`); 2D grid speed × radius + `heatmap_p50` in JSON (`--grid`).
  ```bash
  # random spread only (seed)
  python -m microbe_radiation_model.ensembles --seeds 0,1,2,3,4 --out ensemble.json

  # 3×3 grid, 2 seeds per cell
  python -m microbe_radiation_model.ensembles --grid --v-steps 3 --radius-steps 3 \
    --seeds 0,1 --out grid.json
  ```
- **W-N2 — terminal-events report:** `simulation/terminal_report.py` + `SimulationReport.terminal_events_report`; escaped / collided / arrived counts, median time and survival; Mars pipeline only (not ported to `engine.py`). Bound/escape/Hill classification already existed — do not redo.
- **W-N3 — probability maps:** `web/grid.html` + `charts/heatmap.js`, SVG heatmap `speed × radius` → median survival from JSON (`kind: parameter_grid`).
- **W-N4 — sensitivity:** OAT ±10% in `ensembles/sensitivity.py` + CLI `--tornado` / `--quick`; knobs from `server.py` + Ea, `radiation_surv_coeff`, `hydrolysis_surv_coeff`, GCR k (`run_overrides.py`). The web tornado chart (`charts/tornado.js`) was later replaced on `sensitivity.html` by Morris screening (`charts/morris.js`).

---

## Order

| Step | What | Why |
|---|---|---|
| 0 | W0 ✓ | do not build on WIP / stale README |
| 1 | W1 ✓ | calibration before sweeps |
| 2 | W-N1 → W-N3 → W-N4 ✓ | scientific core for the stage |
| 3 | W-N2 ✓ + W2 | demo credibility |
| 4 | W-P* | after conference / if time remains |
