# Project Guidelines

## Code Style
- Keep changes minimal and consistent with the existing lightweight Flask + scientific Python style used in `controller.py`, `compute.py`, and `cosmology_lib.py`.
- Preserve current module boundaries: form schema in `model.py`, cosmology constants/functions in `cosmology_lib.py`, request handling in `controller.py`.
- Do not introduce large framework swaps unless explicitly requested.
- When touching templates, keep variable names aligned with controller outputs (`rvir`, `vvir`, `tvir`, `result`) as in `templates/view1.html`.

## Architecture
- Main request path: `controller.py` route `/dmhalo` -> validate `InputForm` in `model.py` -> call `compute.compute()` -> render `templates/view1.html`.
- Computation flow: `compute()` calls `rvir_mvir`, `vvir_mvir`, `tvir_vvir`, and `plot_halo_histories`; cosmology helpers come from `cosmology_lib.py`.
- Cosmology parameters are currently global in `parameter.py` and read by `cosmology_lib.py` at module import/reload time.
- Plot output is a base64 data URI generated in `compute.py` and injected directly into `<img src="{{result}}">` in `templates/view1.html`.

## Build and Test
- Install deps: `python3 -m pip install -r requirements.txt`.
- Local run (current code path): `python3 controller.py`.
- Deployment: `gcloud app deploy app.yaml`.
- No automated test suite is present; after edits, at minimum load `/dmhalo`, submit sample values, and confirm the image + scalar outputs render.

## Project Conventions
- Existing code uses `reload(...)` and module-level mutable settings; avoid expanding this pattern.
- Prefer passing values through function arguments for new logic instead of adding new global mutable state.
- Keep scientific unit assumptions explicit in labels and outputs (see units in `templates/view1.html` and formulas in `compute.py`).
- Keep UI changes focused on `templates/view1.html` unless task scope explicitly expands.

## Integration Points
- Web stack: Flask + WTForms (`controller.py`, `model.py`).
- Numeric/plot stack: NumPy + Matplotlib (`compute.py`, `cosmology_lib.py`, `app.yaml`).
- Front-end math rendering: MathJax CDN in `templates/view1.html`.
- App Engine-specific config lives in `app.yaml` and `appengine_config.py`.

## Security
- Input validation is currently only "required" checks in `model.py`; add numeric range guards when changing input behavior.
- Be careful with global parameter mutation in `parameter.py` from request handlers in `controller.py`; this can leak state across requests.
- Avoid adding new dynamic execution or file-system side effects in request paths.
