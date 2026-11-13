# web

The 3D visualizer: a Three.js scene that replays the ejecta swarm produced by the Python
model, plus research pages and offline analysis views for ensemble JSON.

Deployed to GitHub Pages from `main` by `.github/workflows/deploy-pages.yml`.

## Pages

| File | Title |
|---|---|
| `index.html` | Cosmos 3D — the interactive visualizer |
| `research.html` | Research — Could life travel between solar systems? |
| `further_details.html` | Further Details |
| `grid.html` | Survival heatmap (velocity × radius ensemble JSON) |
| `sensitivity.html` | Morris elementary-effects screening (sensitivity JSON) |

All five are declared as Vite build inputs in `vite.config.js`.

## Running it

```bash
npm install
npm run dev        # http://localhost:3000, opens automatically
npm test           # Vitest unit tests
npm run build      # production build into dist/
npm run preview    # serve the built dist/
```

The dev server sets `Cross-Origin-Opener-Policy` and `Cross-Origin-Embedder-Policy`
because the R console embedded in `research.html` runs WebR, which needs
`SharedArrayBuffer`.

## Data contract

Everything the site loads at runtime comes from `public/data/`:

| File | Role | Produced by |
|---|---|---|
| `solar_system.json` | Static scene: bodies, radii, orbital distances, colours | hand-authored, lives only here |
| `cosmos_visualizer_simulation.json` | **Replay of the ejecta swarm** — per-frame positions and per-object properties | the Python model |
| `gamma_radiation_timeseries.json` | Internal gamma field over time | the Python model |
| `rock_radiation_summary.json` | Per-rock dose and temperature summary | the Python model |
| `star_uv_profile.json` | Stellar UV flux versus distance | the Python model |
| `test_replay.json` | Small replay fixture (`?replay=data/test_replay.json`) | hand-authored |
| `grid_sample.json`, `morris_sample.json` | Offline samples for analysis pages | `python -m microbe_radiation_model.ensembles` |

To refresh the model-produced files after a simulation run, from the repository root:

```bash
python tools/export_simulation_to_web.py          # copy what the model has produced
python tools/export_simulation_to_web.py --run    # run the Mars pipeline first, then copy
python tools/export_simulation_to_web.py --check  # report only, write nothing
```

That script never touches the hand-authored files.

A different replay can be loaded without rebuilding, via the query string:

```
index.html?replay=data/test_replay.json
```

## Source layout

| Area | Modules |
|---|---|
| Scene and rendering | `scene.js`, `renderer.js`, `camera.js`, `cameraRoll.js`, `shaderMaterial.js`, `selectionGlow.js`, `sceneScale.js` (display scaling of sizes and distances) |
| Object construction | `objectFactory.js`, `orbitLine.js`, `trailManager.js`, `planetTextures.js` (planet, Sun and ring maps), `fragmentTexture.js` (fragment surfaces generated from rock properties) |
| Simulation playback | `replayController.js`, `replayUI.js`, `animator.js`, `physics.js`, `physicsSync.js`, `orbits.js` (osculating orbits), `orbitalClock.js` (second clock for orbital motion) |
| Asteroid swarm | `asteroidManager.js`, `asteroidUI.js`, `doseColor.js` (fragment colour by absorbed dose) |
| Interaction | `picker.js`, `focusController.js`, `objectSearch.js`, `objectSearchLogic.js`, `infoPanel.js` |
| Headline and provenance | `charts/headline.js`, `charts/headlineBanner.js` (result band), `charts/provenancePanel.js` (reproducibility record) |
| Analysis charts | `liveCharts.js`, `charts/plot.js`, `charts/series.js`, `charts/answerSurface.js` + `charts/answerSurfaceChart.js`, `charts/doseModel.js` (survival recomputed in the browser), `charts/depthProfile.js`, `charts/sameDose.js`, `charts/sizeDecides.js`, `charts/survivalPhase.js`, `charts/trajectoryColor.js`, `charts/popout.js` (charts in their own windows) |
| Science overlays | `uvRadiation.js` |
| Analysis pages | `gridPage.js` + `charts/heatmap.js`, `sensitivityPage.js` + `charts/morris.js` + `charts/morrisChart.js` |
| Presentation mode | `presentation.js` |
| Data | `dataLoader.js`, `api.js` (client for the local simulation server) |
| Run console and menus | `ui/controlPanel.js`, `ui/rangeLog.js`, `ui/paramHelp.js`, `ui/tooltip.js`, `ui/menuBar.js`, `ui/scrollHint.js`, `ui/analysisPage.css`, `ui/theme.css` |
| Entry point | `main.js` |

Paths are resolved through `import.meta.env.BASE_URL`, so the same build works at the
site root during development and under `/lithopanspermia-digital-twin/` on GitHub Pages.

## Tests

`tests/` holds one Vitest file per module for the logic that can be tested without a
browser — geometry construction, replay interpolation, search, camera roll, data loading,
UV shells, the info panel. `vitest.config.js` runs them in the `node` environment.

## Live analysis dock

The panel behind the **ANALYSIS** button is not a separate page — it plots the replay
that is playing, and the curves grow frame by frame alongside the 3D scene. Scrubbing
the timeline rewinds them too.

Four charts, all derived from the replay itself so they can never drift out of sync
with the scene:

| Chart | Source field |
|---|---|
| Surviving microbial fraction | `population_fraction` per frame |
| Distance from the Sun | `positions`, relative to the Sun's own position |
| Speed | magnitude of `velocities` |
| Dust erosion | `radius`, normalised to each fragment's own start |

Two decisions worth keeping:

- **The axes are fixed to the full run.** The curve grows into a stable frame instead
  of the scale sliding under the viewer every step.
- **Erosion is plotted in ppm, not metres.** The absolute loss is ~1e-7 m out of ~2e-2 m;
  on an absolute axis it is a flat line.

`charts/series.js` holds the pure transforms (unit-tested in
`tests/chartSeries.test.js`); `charts/plot.js` holds `liveLinePlot`, which computes the
axes once and then only rewrites each path's `d` attribute per frame — a full SVG
re-render of four charts at replay speed is not affordable.

Adding a chart is a new entry in the `specs` array in `liveCharts.js`. Quantities that
are not per-frame — the per-rock-type comparison, the GCR composition, the internal
gamma dose — live in `gamma_radiation_timeseries.json` and `rock_radiation_summary.json`
and would need a different home, since they are not frame-aligned to the replay.

## What the interface shows, and why

### The headline band

A probability box across the top, with a selectable horizon. It is **not** a confidence
interval and the copy says so: `c_rad` is a fixed number nobody knows, not a sampled one,
so the object is the range of answers consistent with the published literature (Sarma et
al., CHI 2024). Hard ends, no gradient — a soft edge would imply a density that does not
exist.

It also attributes the width: **c_rad accounts for 94 %** of the spread, the entire N-body
run for the rest. That split is scale-free — both terms are linear in time — which a test
pins, because if it ever stopped being true the sentence would have to name its horizon.

### The answer surface

Dose on the horizontal axis, inactivation coefficient on the vertical, both logarithmic so
the contours of `exp(−cD)` are straight lines of slope −1: the eye reads a straight line
far more reliably than a hyperbola. The published chronic band is a shaded **strip, not a
line** — the literature gives a range with no preferred value inside it, and a central line
would invent one. The coefficient slider walks a marker up and down through the contours.

One caveat is stated rather than hidden: hydrolysis is a second channel that moves survival
by about 2 × 10⁻⁴ and cannot live on these two axes, so a point is *drawn* dose-only while
the survival *quoted* for it is the model's real answer.

### The 3D scene

The scene draws the **osculating orbit** each fragment is on — the ellipse it would follow
from here if every perturbation stopped — solved from position and velocity, not
interpolated from samples. Watching those ellipses breathe under the planets *is* the
secular eccentricity pumping that spreads the swarm.

The camera frames the swarm from its own median aphelion. Median rather than maximum, so
the fragment at 31 AU does not shrink everything else to a point; it runs past the edge,
which is honest, because it visibly *is* an outlier.

### The workspace menu

One bar along the top — Figures, Scene, Panels, Analysis — listing every panel by name with
a line saying what it is, a checkmark for what is shown, and a tear-off target beside each
figure. Detached charts open in their own window and stay linked to the replay.

### The run console

Every model parameter, editable, with the numeric field as the primary control and the
slider secondary. The parameter schema is frozen into the bundle at build time, so the full
set of controls renders **with no solver running** — which is how almost everyone will see
it. Each parameter carries a written explanation on hover *and on keyboard focus*: what the
quantity is, what moving it does, and — only where the number is not settled — the published
range and the paper it came from.

### The provenance panel

Every replay carries a SHA-256 digest of its complete parameter set, the resolved seed, the
source commit, whether the tree was dirty, a command that reproduces the run, and a live
audit of which constants are cited. The panel leads with one line: a coloured dot and a
verdict, ordered by how badly each condition undermines the record.

## Sensitivity: why the tornado chart was removed

The sensitivity page used to show a one-at-a-time tornado. It was replaced rather than
annotated, because taking its shipped sample apart showed it was not merely older but
actively misleading. Measured from the file:

- the run lasted **half a year**, with four fragments
- the largest swing it measured was **2.0 × 10⁻⁷**
- **two of the eight knobs moved the result by exactly zero**
- its baseline for the radiation coefficient was 3.16 × 10⁻⁶ 1/Gy — a factor of **7.9 below
  the floor of this model's own published band**

A reader who sees a ±10 % OAT design applied to a coefficient uncertain by a factor of
seventeen has been given a reason to distrust everything else on the screen.

**Morris elementary-effects screening** replaces it, with the conventions checked at the
sources rather than copied from practice, because the two disagree:

| Convention | Source |
|---|---|
| μ\* on x, σ on y, equal unit scale, square area | Morris (1991) *Technometrics* 33(2), Figs 1 & 4; Saltelli et al. (2012) *Chem. Rev.* 112(5) |
| σ/μ\* = 0.1, 0.5, 1 diagonals separating linear / monotonic / interacting | García Sánchez et al. (2014) *Energy and Buildings* 68:741–753 |
| ±2·SEM wedge, SEM = σ/√r — a **significance** test, not a shape classifier | Morris (1991) |
| μ plotted beside μ\*, because μ\* exists so a sign-changing factor does not cancel | Campolongo, Cariboni & Saltelli (2007) *EMS* 22:1509–1518 |
| r(k) = π^(k/2) / (Γ(k/2+1)·2^k) — the fraction of space an OAT design can reach | Saltelli & Annoni (2010) *EMS* 25(12):1508–1517 |

The cost comparison is stated **at equal budget**, not as a slogan: this screening cost 108
model evaluations; a one-at-a-time design over the same eight factors costs the same order,
reaches **1.59 %** of the space, and detects no interactions at all.

What the screening says: simulated time and the radiation coefficient dominate; hydrolysis
is alive, where both its knobs read exactly 0.0 before the freezing cut was removed;
fragment radius is nearly irrelevant. Only fragment radius — the least influential of the
eight — changes sign; every other factor is strictly monotone to within 0.04 %.

## Design decisions worth defending

**The data palette is not Wolfram's, and its limits are stated.** The interface
deliberately resembles Mathematica in chrome, typography and density — but Mathematica's
own default series colours, `ColorData[97]`, fail an accessibility check: slots 2 and 3
separate by ΔE00 **1.8** under protanopia and 12.8 with normal vision.

The six rock classes here are Okabe & Ito, darkened where the published values are too
light. Against **adjacent** pairs every check passes — worst 10.4 under deuteranopia, 18.1
with normal vision — but adjacent pairs are not the test that matters, because two classes
far apart in a list are still plotted on the same axes. Under an **all-pairs** check the
original set had a genuine collision: organic-rich against silicate at ΔE00 **1.2** under
protanopia and 10.6 with normal vision, two solid lines that were for practical purposes
the same colour. Organic-rich is now a sienna, which lifts that pair to 5.9.

What remains on hue alone is stated rather than left to be discovered: icy vs metallic 5.9
(deuteranopia), rubble vs chondrite 13.3 (normal vision), sky vs purple 4.3 (tritanopia,
about one person in ten thousand). Six categorical hues cannot clear the all-pairs
thresholds — that was tested, not assumed; every six-colour set tried failed some pair.

So hue is not the only carrier of class identity: **each class also owns a line style**,
and every member of a class shares it, so colour and dash always agree and either alone
identifies the class. That is a correction. The dash used to encode the member *index*
within a class, with the same patterns reused across classes — which left basalt and
organic-rich both drawn solid, so for exactly the pair whose hues are closest, the
redundant channel was identical too. The secondary encoding was backing up the wrong
thing. What is given up is telling olivine from enstatite at a glance, which is the right
trade: their hues never differed either, because they are one class, and the exact rock
type is named in the tooltip and the legend.

Selection is deliberately **not** blue — the navy accent already owns that register — and
not the obvious magenta either: the first candidate separated from the icy rock class by
ΔE00 **4.7** with normal vision, so a selected fragment and an icy one were the same
colour. The violet that replaced it clears 15.8.

**Light ground, restrained navy, dark scene.** Everything a reader lifts into a slide
lands on white, and a projector in a lit room destroys exactly the low tones a dark scene
of small dots depends on. The accent is a muted navy rather than an orange: orange is a
*warning* colour in instrument conventions, and spending it on ordinary headings and links
leaves nothing louder for an actual caution — navy is the register scientific publishing
already uses for structure and reference, and it keeps the warm end of the spectrum free
for **data**, where the vermillion now lives and nowhere else. The neutrals are cooled to
match, because a warm grey beside a navy accent reads as two unrelated decisions.

The 3D view stays dark as an inset with its own ground, so the rule reads in two seconds:
the dark rectangle is the observation, everything around it is the instrument.

Every colour is measured against the ground — `--ink` 12.00:1, `--accent` 8.06:1, `--warn`
4.71:1, `--bad` 6.90:1 — and `--accent-lit` exists **only** for the dark inset, where it
reads 5.78:1 while the navy itself would read 2.30:1.

**Multiplicative notation.** Uncertain coefficients are written `1.0e-4 ×/ 4.1 1/Gy`, not
`±`. A quantity known to within a factor is not symmetric on a linear scale (Limpert, Stahel
& Abbt 2001, *BioScience* 51(5):341–352). The centre is the **geometric** mean of the
endpoints, which reproduces 2.5e-5 and 4.3e-4 exactly.

**Explanations are not `title` attributes.** The HTML specification discourages `title`
because user agents do not expose it accessibly; it never appears on keyboard focus, cannot
be hovered so a citation inside it is unclickable, and does not appear at all on touch. The
replacement satisfies WCAG 2.2 SC 1.4.13 — dismissable, hoverable, persistent — and each
of those three is tested rather than asserted.

## Backlog

- Bottom-corner status bar showing the current radiation level for the focused object.
