# FfM_Water_Management4Trees

# DWD Urban Heat Island → Frankfurt UHI Grid (KNIME workflow)

End-to-end ETL workflow that ingests the German Weather Service (DWD) urban heat island raster dataset, filters it to the Frankfurt am Main area, aggregates it to a monthly mean per grid cell, and exports it as a flat CSV ready for downstream visualisation.

Built as part of the **Life of a Tree** portfolio, an urban tree water management dashboard for Frankfurt that combines tree cadastre data, live weather, and urban heat data to model irrigation needs and cooling output at the street-tree level.

## Data source

**Dataset:** DWD HOSTRADA, High-Resolution Hourly Raster Dataset, urban heat island intensity
**Source:** [opendata.dwd.de — climate_environment/CDC/grids_germany/hourly/hostrada/urban_heat_island_intensity](https://opendata.dwd.de/climate_environment/CDC/grids_germany/hourly/hostrada/urban_heat_island_intensity/)
**Format:** NetCDF (.nc), 1 km × 1 km grid, EPSG:3034 projection, UTC timestamps
**Coverage used:** April 2026 (one monthly file, 720 hourly timesteps)

The UHI intensity values are derived from the HOSTRADA dataset using Copernicus CORINE Land Cover 2018 as the urban land cover input.

## Workflow

```
Python Source (legacy)  ─►  CSV Writer
        │
        └── xarray reads .nc file
            → flatten to dataframe
            → filter to Frankfurt bbox (lat 50.02 to 50.23, lon 8.47 to 8.83)
            → groupby (lat, lon) → mean UHI intensity
            → output ~575 grid cells × 1 mean value
```

**Nodes:**
`Python Source (legacy)` reads the NetCDF via `xarray`, applies spatial filter and monthly aggregation.
`CSV Writer` exports the flat table.

**Why Python Source?** KNIME free desktop has no native NetCDF reader. The Python Source (legacy) node bridges this with a small `xarray` + `pandas` script while keeping the workflow visual and reproducible.

## Output

`frankfurt_uhi.csv`, ~575 rows, one row per 1 km² Frankfurt grid cell.

| Column | Description |
|---|---|
| `lat` | Latitude of grid cell centroid (WGS84) |
| `lon` | Longitude of grid cell centroid (WGS84) |
| `uhi` | Monthly mean UHI intensity in Kelvin (delta vs. surrounding rural baseline) |

This CSV feeds directly into the Life of a Tree Lovable app as the UHI overlay layer, allowing district-level heat stress to be cross-referenced against tree cooling capacity.

## Reproducing locally

1. Download a HOSTRADA UHI monthly NetCDF file from the DWD link above
2. Open `dwd_uhi_frankfurt.knwf` in KNIME Analytics Platform (free desktop, 5.x)
3. In **Preferences → KNIME → Python (legacy)** set Python 3 to **Manual** mode pointing to a Python 3.9+ install with `xarray`, `netCDF4`, and `pandas` installed
4. Update the file path in the `Python Source (legacy)` node to point to your downloaded `.nc` file
5. Execute

## Stack

| Layer | Tool |
|---|---|
| ETL | KNIME Analytics Platform (free desktop) |
| NetCDF parsing | Python 3.9 + xarray + netCDF4 |
| Data source | DWD Climate Data Center (CDC) open data |
| Downstream consumer | Life of a Tree (Lovable app) |
