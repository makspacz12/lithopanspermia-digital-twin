# legacy_modules

The actual store of inactive, historical or experimental artifacts.

## Modules

- `shielding_legacy.py`
  - older radiation attenuation model for a homogeneous rock
  - has no biological core, so it works as a simple reference model
- `__init__.py`
  - exports selected historical elements for compatibility imports

## Technical status

Nothing here takes part in the default demo or simulation run. It is kept as reference
and as a starting point for further experiments.

> `XCOM.exe` used to live here — despite the extension it was a ZIP archive, and it was
> removed during repository cleanup. XCOM is NIST's photon cross-section tool; download
> it from NIST directly if the attenuation-coefficient work resumes.
