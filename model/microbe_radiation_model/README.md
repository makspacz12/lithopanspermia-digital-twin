# microbe_radiation_model

This package holds the complete pipeline:

1. physical and material definitions,
2. radiation and shielding calculations,
3. thermal, chemical and biological state,
4. time integration with REBOUND,
5. runnable demos and reports.

The layers and every module in them are listed in [MODULE_CATALOG.md](MODULE_CATALOG.md);
the physics and data flow are described in [TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md).
Run commands are in [../../docs/en/RUNNING.md](../../docs/en/RUNNING.md).

## Clean package root

At runtime only the following live directly under `microbe_radiation_model/`:

- `__init__.py` — the public API
- `data_store.py` — JSON persistence
- `asteroid_state.py` — per-asteroid mutable state
- the server, CLI and provenance modules (`server.py`, `cli.py`, `provenance.py`, `run_overrides.py`)
- documentation (`README.md`, `MODULE_CATALOG.md`, `TECHNICAL_DOCUMENTATION.md`)
- the layer directories

The old flat aliases that used to sit at this level were removed from the runtime
(they remain in git history).
