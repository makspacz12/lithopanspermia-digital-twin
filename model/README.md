# model

The Python side of the lithopanspermia digital twin: orbital transport, radiation,
thermal and chemical state, and the microbial survival function.

It is the merged result of two branches that were developed in parallel:

- the `Hack4Sages_project` line contributed the orbital and dynamical layer —
  REBOUND, Mars ejecta, radiation pressure and dust erosion;
- the `Hack4Sages_maks` line contributed the temperature model, hydrolysis, the JSON
  data store and the structured documentation.

## What is canonical

- The active runtime is `microbe_radiation_model/`.
- The main entry point is `run.py`.
- The canonical data format for visualization is JSON in `microbe_radiation_model/data/`.
- JSON is the only active export path for the visualizer.

## Files at this level

| Item | Role |
|---|---|
| `microbe_radiation_model/` | **The runtime package.** Its layers (`physics/`, `radiation/`, `thermal/`, `chemistry/`, `biology/`, `impacts/`, `erosion/`, `simulation/`, …) are listed module by module in [MODULE_CATALOG.md](microbe_radiation_model/MODULE_CATALOG.md). |
| `run.py` | Main entry point. Prints a runtime report for the static and the connected pipeline. |
| `requirements.txt` | Dependencies for the base runtime plus the optional extensions (`rebound`, `reboundx`, `spiceypy`). |
| `requirements-ci.txt` | Pinned dependencies used by the GitHub Actions test workflow. |
| `nearest_50_gaia.csv` | Gaia catalog of the 50 nearest stars, used to build the interstellar environment. |
| `tests/` | Python test suite (`python -m unittest discover tests`). |

Old notebooks and legacy modules live in the repository-level `../archive/`.
Nothing in `microbe_radiation_model/` imports them.

## Quick start

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python run.py
```

Other entry points:

```bash
python -m microbe_radiation_model.demos.run_radiation_demo     # static, no REBOUND
python -m microbe_radiation_model.demos.run_simulation         # connected pipeline
python -m microbe_radiation_model.demos.run_mars_impact_demo   # Mars ejecta
python -m microbe_radiation_model.demos.run_mars_pipeline      # full pipeline + JSON export
```

The complete command table is in [../docs/en/RUNNING.md](../docs/en/RUNNING.md), and the script that
feeds the output into the 3D visualizer is `../tools/export_simulation_to_web.py`.

`run.py` also works in an environment without the full scientific stack. When
`rebound` is not importable it falls back to a **static radiation mode** — the report
then says so explicitly, and the "static" and "connected" outputs become identical.

## Exports for visualization

Written into `microbe_radiation_model/data/`:

| File | Contents | Committed? |
|---|---|---|
| `star_uv_profile.json` | Stellar UV flux versus distance | yes |
| `solar_system_horizons_cache.json` | Cached JPL Horizons state vectors, so runs work offline | yes |
| `gamma_radiation_timeseries.json` | Internal gamma field over time | generated |
| `rock_radiation_summary.json` | Per-rock dose / temperature summary | generated |
| `cosmos_visualizer_simulation.json` | Replay file for the 3D viewer | generated |

## External data sources

| Source | Fetched by | Notes |
|---|---|---|
| Gaia archive | `simulation/gaia_catalog.py`, `demos/fetch_gaia_catalog.py` | Refreshes `nearest_50_gaia.csv`; requires `astroquery` |
| JPL Horizons | `simulation/solar_system.py` | Planetary state vectors; cached to JSON |
| NAIF SPICE kernels | `simulation/solar_system.py` | `de440.bsp`, `naif0012.tls`, `gm_de431.tpc`, `pck00010.tpc` downloaded into `kernels/` on first use (gitignored) |

## Two fixed pitfalls worth knowing about

Both of these used to silently zero out the radiation-to-biology chain. They are fixed,
but the underlying traps are easy to reintroduce:

1. **Fragment radius.** `simulation/config.py::default_material_config()` now uses
   `DEFAULT_FRAGMENT_RADIUS_M` (0.5 m). It must **not** read `rock.radius_m` from the
   catalog — for source-derived variants that field is the radius of the *reference
   body* (asteroid 4 Vesta, 261 385 m for `basalt_vtype`), five orders of magnitude
   larger than the 0.001-5 m fragments the impact model samples.
2. **`Material.k` is a mass attenuation coefficient [m²/kg]**, used as
   `exp(-k·ρ·x)`. It is not thermal conductivity [W/(m·K)]. The rock's thermal
   conductivity now lives in `SimulationMaterialConfig.rock_thermal_conductivity_w_mk`.

## Documentation

- [microbe_radiation_model/MODULE_CATALOG.md](microbe_radiation_model/MODULE_CATALOG.md) — one line per module, plus the high-level data flow
- [microbe_radiation_model/TECHNICAL_DOCUMENTATION.md](microbe_radiation_model/TECHNICAL_DOCUMENTATION.md) — full physics and data-flow reference
- each layer directory has a short `README.md` of its own
