# RailCAST

**Rail station catchments, road accessibility, and scenario exploration for North Carolina.**

RailCAST is a research demonstration developed by Tianyang Chen at UNC Charlotte's Center for Applied GIScience (CAGIS). It combines a discrete-choice station model, a connected road network, Census population weights, and NCDOT survey summaries in an interactive React dashboard.

This repository contains the current demo as a standalone **React + TypeScript + Vite** application. No ChatGPT account, hosted backend, API key, or database is required to run it. It is a research prototype, not an official NCDOT product or a validated ridership forecast.

## Run locally

Install Node.js 22.13 or newer, then:

```bash
git clone https://github.com/TianyangChen357/demo_RailCAST.git
cd demo_RailCAST
npm ci
npm run dev
```

Open the localhost URL printed by Vite. The bundled data are about 116 MB; the first map load downloads the required state layers and route matrices. Local streets load on demand for the selected station. The optional online street basemap requires internet access; bundled roads and model data work locally.

```bash
npm test          # Model, calibration, probability, and survey checks
npm run build    # TypeScript validation and production build
npm run preview  # Preview dist/ locally
```

The `dist/` directory can be served by a static web host. If serving under a subdirectory, set `BASE_PATH` during the build, for example `BASE_PATH=/demo_RailCAST/ npm run build` on macOS/Linux. On PowerShell use `$env:BASE_PATH='/demo_RailCAST/'; npm run build`. A public repository does not automatically enable GitHub Pages.

## Explore the dashboard

| View | What to try |
| --- | --- |
| **Statewide** | Activate a survey-requested city such as Winston-Salem. Inspect its highlighted area, transfers from neighboring stations, and change relative to the baseline. |
| **City & station** | Select a station; adjust access time, frequency, fares, parking, or access improvements. Switch to the illustrative train-versus-car view. |
| **Survey trends** | Compare the observed 2014, 2016, 2019, 2023, and 2024 survey summaries. Missing observations remain missing. |

Station numbers, colors, shared boundaries, and a focus mode clarify ownership. **Shared areas** identifies cells above the probability threshold for more than one station. **Map layers → 2020 Census tract evidence** shows the population source units. **Data & model** explains the data and calibration.

## Data and model

- 12 baseline stations and 36 survey-requested candidate locations; station locations are approximate.
- 86,618 clipped cells: 55,357 at 500 m near station cities and the remainder at 2 km.
- 2,672 Census tracts, with 10,439,388 residents from the 2020 Census. Population is allocated uniformly within tracts; fine cells are not individual household observations.
- OpenStreetMap snapshot **2026-09-27**, accessed through Geofabrik. The graph contains about 12.13 million nodes and 24.32 million directed edges.
- Precomputed shortest road travel times to all 48 alternatives. One-way/access tags are honored; travel speeds, connectors, and several network details remain assumptions.
- An explicit outside alternative prevents a remote location from receiving 100% station probability solely because only one station remains available.
- Existing-station constants are balanced against the 2024 survey's NC departure-response shares, with Census population as origin weights. At the default settings, mean absolute share error decreases from about **4.62 to 0.10 percentage points**.

That final metric is **in-sample aggregate fit**, not independent validation. Time/cost sensitivity, the outside-option rate, candidate-station attractiveness, and train/car mode shares have not been behaviorally estimated. The survey sample is not annual ridership. Zero catchment area at a chosen threshold does not imply no service or no passengers. Source-year differences, route limitations, and all parameters are documented in the app and [method notes](public/data/network-methods.txt).

## Reproduce geographic preprocessing

The ready-to-run derived data are included; preprocessing is optional and considerably more demanding than running the app. The reference build used Python 3.12 and roughly 10 GB of available RAM. Allow several GB of disk space for the source extract and intermediate routing graph.

```bash
python -m venv .venv
# Activate the environment for your operating system.
pip install -r requirements-geo.txt
python scripts/download-network-inputs.py --cache data-cache
python scripts/prepare-network.py --cache data-cache
```

`download-network-inputs.py` fetches Census geometries and the latest Geofabrik extract, and exports the station definitions from TypeScript. `prepare-network.py` builds the routing graph, adaptive grid, population allocation, route matrices, and calibration metadata. Its cached inputs must stay consistent; use a new cache directory when changing the source snapshot, station coordinates, or preprocessing logic.

The OSM `latest` URL changes over time. Re-running with a newer extract is a new model snapshot, not a bit-for-bit reconstruction of the bundled September 2026 outputs. The exact computed matrices and snapshot metadata are preserved in this repository. `prepare-rail-geography.py` documents the original NCDOT display-layer preparation; its legacy 5 km grid is retained for provenance and is not used by the current model.

## Project structure

```text
app/                     Dashboard, map, model, survey data and styles
components/ui/           UI primitives used by the dashboard
public/data/             Ready-to-run geographic data and route matrices
public/sources/          Public source citations and methodology summary
scripts/                 Model checks and geographic preprocessing
vendor/                  Third-party styles and their license notice
```

## Sources and permissions

See [DATA_SOURCES.md](DATA_SOURCES.md) and [LICENSING.md](LICENSING.md). OSM-derived data retain ODbL attribution and licensing. The source survey PDF and working proposal are not redistributed; the public app links to citation and methodology pages instead of private document links. No individual survey records or personal travel histories are included.

Original project code is publicly available for inspection; an additional open-source license has not been selected. Third-party code and data retain their own terms.
