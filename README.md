# Lithopanspermia Digital Twin

**A computational model of interstellar microbial transfer, from impact ejection to
arrival dose.**

🥇 **First place worldwide, Hack4Sages** · 🇨🇭 **Selected for presentation at ETH Zürich**

Live demo: **https://makspacz12.github.io/lithopanspermia-digital-twin/**

---

## Main result

Over the 3 000-year run the surviving microbial fraction stays between **0.77 and 0.97**,
so on that timescale the uncertainty in the biology barely shows. Extrapolated
to the transfer times the literature actually discusses, the same published band for the
one uncertain biological coefficient opens to **43 orders of magnitude at 1 Myr** and 432
at 10 Myr. Sterilisation to N/N₀ = 10⁻⁶ arrives after **133 kyr to 3.0 Myr**, against
transfer times of tens of Myr.

**The model's answer is, to first order, a statement about one coefficient nobody has
pinned down.** Across the swarm, survival correlates with the sampled `c_rad` at
**r = −0.993**. For this reason the interface puts this coefficient first and lets you
change it and see the result update.

---

## About

Research into the origin of life has largely asked how biology arises *within* a single
environment. Lithopanspermia asks a different question: whether living material can be
**redistributed between planetary systems** by ordinary astrophysical processes and, if
so, under what conditions it could still be viable on arrival.

The conclusion of the hypothesis is speculative, but each of its premises is
independently established: hypervelocity impacts eject competent rock at escape velocity;
some fraction of that ejecta leaves the system entirely; microorganisms have documented
tolerances to vacuum, shock and radiation; and the dose accumulated in transit can be
computed rather than assumed.

This project is an end-to-end model that follows a fragment from ejection to arrival
and tracks what happens to the life inside it. It couples an N-body gravitational
transport layer to a physically explicit survival calculation, so the outcome for any
fragment can be traced back to the mechanisms that produced it.

---

## The survival function factorises exactly

Survival in this model is

```
N/N₀ = exp( −c_rad · D_cum  −  c_hyd · H_cum )
```

verified against the shipped replay to a worst deviation of **1.2 × 10⁻¹⁵**. Two numbers
per fragment therefore settle the answer completely: the dose it accumulated, and the
inactivation coefficient it was assigned.

This has three consequences for the design:

1. **The rest of the model serves to compute the first of those two numbers.** The
   N-body integration, the thermal model, radiation pressure, dust erosion and the
   radionuclide chain all feed into the dose a fragment accumulates.
2. **The biological answer can be recomputed in the browser instantly, for any
   coefficient, with no new simulation.** This is why the coefficient is a live control
   rather than a fixed input.
3. **The main figure is not a time series.** It is the plane spanned by those two numbers,
   with iso-survival contours across it and the swarm placed on it. This is the *answer
   surface*, the first figure in the analysis dock.

---

## What the interface shows

A headline band with the range of answers the literature allows (explicitly *not* a
confidence interval), the **answer surface** where that range lives, a 3D scene of the
swarm on its osculating orbits, a run console that renders every model parameter even
with no solver running, and a provenance panel on every replay. What each element shows
and why it is drawn that way is in [web/README.md](web/README.md#what-the-interface-shows-and-why).

---

## Scientific practice

- **Every run carries a provenance record** (seed, parameter digest, commit, package
  versions). Runs reproduce bit-for-bit.
- **The physics is tested.** 290 Python and 666 JavaScript tests, compared where possible
  against published values rather than the code's own earlier output, so two compensating
  errors cannot hide each other.
- **Uncertain constants are listed explicitly.** Every output file includes an audit of
  which coefficients are cited and which are not.
- **Numbers quoted in code comments were checked.** Where a comment turned out to be
  wrong, it now gives the measured value and notes the earlier error.

---

## Repository layout

```
.
├── model/       Python simulation — transport, radiation, thermal, chemistry, biology
├── web/         3D visualizer (Three.js + Vite), deployed to GitHub Pages
├── analysis/    Empirical survival-function work (R + Python) and its figures
├── tools/       Scripts that connect the three above
├── docs/        docs/en/ — RUNNING.md (install & run), conference notes; docs/pl/ — Polish docs
├── archive/     Old, unused material kept for reference (not part of the runtime)
├── presentation/  Conference slide plan and screenshots
└── README.md    this file
```

| Part | Language | Entry point |
|---|---|---|
| [`model/`](model/) | Python 3.12+ | `python run.py` |
| [`web/`](web/) | JavaScript (Three.js, Vite) | `npm run dev` |
| [`analysis/`](analysis/) | R + Python | `Rscript radiation_to_survival.R` |
| [`tools/`](tools/) | Python | `python tools/export_simulation_to_web.py` |

The visualizer can drive the model directly: start
`python -m microbe_radiation_model.server` and the **Run console** picks parameters,
launches a real REBOUND run, shows its progress and plays the result. Without the solver
the page falls back to the bundled replay, with every control still rendered and editable.

---

## Quick start

**Simulation:**

```bash
cd model
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python run.py
```

**Visualizer:**

```bash
cd web
npm install
npm run dev          # http://localhost:3000
npm test             # Vitest unit tests
```

Sensitivity screening, parameter sweeps, feeding a fresh simulation into the visualizer,
re-freezing the parameter schema and every other command:
[docs/en/RUNNING.md](docs/en/RUNNING.md).

---

## How the model works

```
stellar mass → luminosity → flux at the rock surface
    → Beer-Lambert attenuation through rock and biological core → local flux
    → cumulative dose (× time) → surviving microbial fraction
```

Temperature is computed in parallel (surface from radiative equilibrium, interior from
radiogenic heating) and drives the hydrolysis rate.

| Layer | Where |
|---|---|
| Orbital transport (REBOUND), impact ejecta, dust erosion | `model/microbe_radiation_model/simulation/`, `impacts/`, `erosion/` |
| Stellar radiation, galactic cosmic rays, shielding, internal gamma | `model/microbe_radiation_model/radiation/` |
| Surface and interior temperature, radiogenic heat, hydrolysis | `model/microbe_radiation_model/thermal/`, `internal_heat/`, `chemistry/` |
| Survival function | `model/microbe_radiation_model/biology/` |
| Rock properties, cited to source | `model/microbe_radiation_model/materials/rocks/` |
| Osculating orbits for the scene | `web/src/orbits.js` |

---

## What the 3 000-year reference run actually contains

Numbers taken from the shipped replay (`web/public/data/cosmos_visualizer_simulation.json`).
The page itself opens on the 100 kyr run, where the phase
diagram and "same dose, different fate" have something to show; the 3 000-year run is one
click away under **Analysis**.

| | |
|---|---|
| Fragments | 14, radii **1.3 mm to 57.5 mm** |
| Duration | 3 000 years, 151 frames, 73 bodies (Sun, 8 planets, 50 real Gaia stars, 14 fragments) |
| Accumulated dose | 553 – 726 Gy |
| Surviving fraction | 0.775 – 0.971 |
| Fates | **bound 14 · unbound 0 · arrived 0** |

---

## Known questions in the physics

- **Nothing in a typical swarm is shielded.** Fragments are millimetres to centimetres,
  while the cosmic-ray attenuation length is about half a metre.
- **Fragments do not escape on the timescales the demo runs.**
- **Survival times are far shorter than interstellar transit times.** This is what the model predicts, not a bug.
- **Internal U/Th/K decay is negligible:** 0.042 % of the dose.
- **The rock thermal model is isothermal**, and **the 20-year output step aliases the orbits**.
- **`hydrolysis_surv_coeff = 1200` has no cited source**; every other coefficient is cited.

Each point, with numbers and the status of every coefficient, is in
[TECHNICAL_DOCUMENTATION.md §7](model/microbe_radiation_model/TECHNICAL_DOCUMENTATION.md#7-assumptions-and-limitations).

---

## Documentation

- [docs/en/RUNNING.md](docs/en/RUNNING.md) — installation and every run command
- [model/README.md](model/README.md) — what each item in the model is for, exports and data sources
- [model/microbe_radiation_model/MODULE_CATALOG.md](model/microbe_radiation_model/MODULE_CATALOG.md) — one line per module
- [model/microbe_radiation_model/TECHNICAL_DOCUMENTATION.md](model/microbe_radiation_model/TECHNICAL_DOCUMENTATION.md) — physics and data-flow reference
- [web/README.md](web/README.md) — visualizer architecture, data contract, the interface, the Morris sensitivity screening that replaced the tornado chart, and the design decisions (palette, notation, tooltips)
- [analysis/README.md](analysis/README.md) — where the survival coefficients come from
