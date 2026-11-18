# Urban form and heatwave population exposure in Global South cities

Code for the paper *"Urban form concentrates heatwave exposure and pinpoints cooling priorities in Global South cities"*.

The analysis covers 35 cities over 2016–2023 on the 0.1° ERA5-Land grid (652 grid cells, of which 572 enter the
models). It identifies heatwaves, builds a grid-cell panel of exposure, building morphology and urban-context
variables, estimates morphology–exposure associations with double machine learning (DML), and compares rules for
siting a fixed budget of cooling interventions in a counterfactual simulation.

## Repository structure

| Folder | Notebook | Content |
|---|---|---|
| `01_heatwave` | `01_heatwave_identification` | Daytime-only, nighttime-only and compound heatwaves under five definitions |
| `02_variables` | `01_population_exposure` | Population within the urban study area and heatwave exposure |
| | `02_building_morphology` | Building density and mean building height |
| | `03_terrain` | Elevation and slope |
| | `04_climate_controls` | Relative humidity, precipitation, cooling degree days |
| | `05_vegetation_water_impervious` | NDVI, water coverage, impervious surface cover |
| | `06_coast_cbd_lcz` | Distance to coast, distance to CBD, local climate zones |
| `03_panel` | `01_merge_exposure_climate` | Base panels (exposure + climate) for the five heatwave definitions |
| | `02_final_panel` | Final analysis panel and sample flags |
| `04_dml` | `01_dml_repeated_crossfit` | DML estimates (all samples, definitions and covariate sets) |
| `05_shap` | `01_xgboost_shap` | XGBoost exposure models, cross-validation and SHAP values |
| `06_simulation` | `01_siting_simulation` | Siting rules across 18 settings |
| | `02_gain_curves` | Avoided exposure by number of cooled cells |
| `07_figures` | `01_fig1_exposure_patterns` | Fig. 1 and the statistics of the first Results section |
| | `02_fig2_dml` | Fig. 2 |
| | `03_fig3_siting` | Fig. 3 |
| | `04_supplementary_figures` | Supplementary Figs. S1–S10 |
| | `05_supplementary_tables` | Supplementary Tables S1–S4 |

The folder `data/` holds the small input files that most notebooks read:

| Path | Content |
|---|---|
| `data/study_grid_cells.csv` | The 652 study grid cells: city, cell id and centre coordinates |
| `data/city_attributes.csv` | Region and Köppen climate zone of the 35 cities |
| `data/grids/{city}_era5_grid.gpkg` | Grid-cell polygons of each city |
| `data/smod_0514.shp` | Urban study boundary of each city (from GHS-SMOD) |
| `data/cbd/cbd.shp` | Central business district points (marked manually) |
| `data/temperature/` | Daily maximum and minimum 2 m temperature of the study grid cells, 2016–2025 (one NetCDF file per city, from ERA5-Land), and the manifest listing the files with their SHA-256 checksums |
| `data/natural_earth/` | Natural Earth 1:110m land and coastline (base map of Fig. 1) |

The two CSV tables are part of the repository. The other files are distributed with the data deposit of the
paper (see its Data availability statement) and are placed in `data/` as shown above.

The large third-party datasets are read from `data/raw/`, one subfolder per dataset (see *Input data* below).
They are not part of the repository and have to be obtained from their providers.

Results are written to `outputs/`, one subfolder per step:

| Subfolder of `outputs/` | Written by |
|---|---|
| `heatwave` | `01_heatwave/01_heatwave_identification` |
| `population_exposure` | `02_variables/01_population_exposure` |
| `building` | `02_variables/02_building_morphology` |
| `terrain` | `02_variables/03_terrain` |
| `climate`, `cdd` | `02_variables/04_climate_controls` |
| `vegetation_water`, `impervious` | `02_variables/05_vegetation_water_impervious` |
| `coast_distance`, `cbd_distance`, `lcz` | `02_variables/06_coast_cbd_lcz` |
| `panel_exposure_climate` | `03_panel/01_merge_exposure_climate` |
| `final_panel_full`, `final_panel` | `03_panel/02_final_panel` (first and second cell) |
| `dml` | `04_dml/01_dml_repeated_crossfit` |
| `shap` | `05_shap/01_xgboost_shap` |
| `simulation`, `gain_curves` | `06_simulation` |
| `figures/<figure>`, `tables` | `07_figures` |

Each step writes to its own fixed subfolder and later steps read from there; running a step again replaces its
earlier results. Of `outputs/`, only the six analysis panels in `outputs/final_panel/` are part of the repository:

| File | Heatwave definition | Read by |
|---|---|---|
| `panel_final_main_local_p90_with_p95_2016_2023.csv` | Local 90th percentile, with the local 95th-percentile outcomes added | `05_shap`, `06_simulation`, `07_figures` |
| `panel_final_local_p90_2016_2023.csv` | Local 90th percentile | `04_dml` |
| `panel_final_local_p95_2016_2023.csv` | Local 95th percentile | `04_dml` |
| `panel_final_city_p90_2016_2023.csv` | City-wide 90th percentile | `04_dml` |
| `panel_final_city_p95_2016_2023.csv` | City-wide 95th percentile | `04_dml` |
| `panel_final_fixed_tx35_tn25_2016_2023.csv` | Fixed thresholds (Tmax 35 °C, Tmin 25 °C) | `04_dml` |

With them, `04_dml` and `05_shap` can be run directly. The other result tables are distributed with the data
deposit of the paper and are placed in the subfolders shown above.

## Order of execution

Run the folders in numerical order and, within a folder, the notebooks in numerical order. Within a notebook, run
the cells from top to bottom.

1. `01_heatwave` → heatwave days per grid cell and year.
2. `02_variables` → one output table per variable group. Notebook `01` needs the heatwave output; the others only
   need the grid and the raw data.
3. `03_panel` → the analysis panels in `outputs/final_panel/`, which every later notebook reads.
4. `04_dml`, `05_shap`, `06_simulation` → independent of one another.
5. `07_figures` → reads the outputs of steps 3 and 4.

## Running the code

- Each cell is self-contained: it has its own imports and configuration block and does not use variables defined
  in other cells.
- **Paths.** Paths are set in the configuration block at the top of each cell. All paths are relative to the
  notebook folder (`../data/...`, `../outputs/...`), which works when a notebook is run from its own folder, as
  Jupyter does by default. Place the third-party datasets in the subfolders of `data/raw/` listed under
  *Input data*, or edit the paths to point to your copies.
- **Outputs.** Cells never modify the files in `data/`. Each step writes to its fixed subfolder of `outputs/`
  (created if it does not exist); no path has to be edited between steps.
- **Not included.** Retrieval and pre-processing of the raw data (downloading ERA5-Land, exporting and mosaicking
  the satellite-derived rasters) are not part of this repository; the notebooks start from local copies of the
  datasets listed below.
- Stored notebook outputs have been cleared.

## Input data

| Data | Used for | Folder |
|---|---|---|
| ERA5-Land daily statistics, 2 m temperature (daily maximum and minimum) | Heatwaves, cooling degree days | `data/temperature` |
| ERA5-Land monthly means (doi:10.24381/cds.68d2bb30) | Relative humidity, precipitation | `data/raw/era5_land_monthly` |
| WorldPop 1 km age–sex population | Population and exposure | `data/raw/worldpop` |
| GHS-SMOD | Urban study area | `data/smod_0514.shp` |
| Open Buildings 2.5D Temporal | Building density and height | `data/raw/open_buildings` (one subfolder per city) |
| Dynamic World | Impervious surface cover, water coverage | `data/raw/impervious`, `data/raw/water` |
| Landsat NDVI | Vegetation | `data/raw/ndvi` |
| Copernicus GLO-30 DEM (doi:10.5270/ESA-c5d3d65) | Elevation and slope | `data/raw/dem` |
| Global map of local climate zones, v3 (doi:10.5281/zenodo.8419340) | LCZ shares | `data/raw/lcz` |
| OpenStreetMap coastlines (osmdata.openstreetmap.de, data of 25 February 2026) | Distance to coast | `data/raw/coastlines` |
| Central business district points (marked manually on satellite imagery) | Distance to CBD | `data/cbd` |
| Natural Earth 1:110m land and coastline | Base map of Fig. 1 | `data/natural_earth` |

The file names expected in each folder are given in the configuration block of the cell that reads it. See the
Data availability statement of the paper for access details.

## Software

Python 3.13. Package versions used for the analysis are listed in `requirements.txt`.

```bash
pip install -r requirements.txt
```

## Figures and tables

| Item | Notebook | Output folder |
|---|---|---|
| Fig. 1 | `07_figures/01_fig1_exposure_patterns` (panels drawn separately and assembled in vector-graphics software) | `outputs/figures/fig1` |
| Fig. 2 | `07_figures/02_fig2_dml` | `outputs/figures/fig2` |
| Fig. 3 | `07_figures/03_fig3_siting` | `outputs/figures/fig3` |
| Supplementary Figs. S1–S10 | `07_figures/04_supplementary_figures` (S7 and S8 from the SHAP values written by `05_shap/01_xgboost_shap`) | `outputs/figures/figS1`, `figS2_S3`, `figS4_S10`, `figS5_S6`, `figS7_S8`, `figS9` |
| Supplementary Tables S1–S4 | `07_figures/05_supplementary_tables` | `outputs/tables` |

## Licence

The code is released under the MIT Licence (see `LICENSE`). The input datasets remain subject to the licences of
their providers.
