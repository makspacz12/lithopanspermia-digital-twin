# archive

Old and unused material, kept for reference. Nothing here is imported, loaded or
tested by the model or the website.

## Contents

| Path | Contents |
|---|---|
| `concepts/amuse.txt` | Analysis of a possible AMUSE integration as a multi-solver layer |
| `concepts/atm.txt` | Analysis of a possible ATM integration for asteroid thermal models |
| `concepts/README.md` | Short index of the concept notes |
| `model/legacy_modules/` | Old uniform-sphere shielding model (superseded by `model/microbe_radiation_model/radiation/shielding_model.py`) |
| `model/environment.ipynb` | The original working notebook (Gaia download, Solar System build); superseded by the `microbe_radiation_model` package |
| `web/tornado.js`, `web/tornado.test.js` | One-at-a-time tornado chart, replaced on `sensitivity.html` by Morris screening (`web/src/charts/morris.js`). Kept so old CLI `--tornado` output can still be read |
| `web/tornado_sample.json` | Sample input for the tornado chart |
| `web/simulation_template.json` | Hand-written template of the replay JSON format |

## Usage status

- nothing here is part of the runtime, the build or the test suites
- `web/tornado.test.js` is not picked up by Vitest from here; move it back to
  `web/tests/` together with `tornado.js` (into `web/src/charts/`) to revive the chart
- the notebook writes `nearest_50_gaia.csv` and SPICE kernels into the directory it is run from
