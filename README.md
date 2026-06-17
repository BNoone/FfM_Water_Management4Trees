# 🌳 Frankfurt: Tree Water Management Dashboard

**Frankfurt has 161,000 street trees. On a hot June day, a city gardener has limited water. Which trees do you water first?**

This project answers that question by combining tree biology, live weather data, and urban heat island science into a single priority score - visualised on an interactive map.

🔗 **Live app:** [frankfurt-tree-watch.lovable.app](https://frankfurt-tree-watch.lovable.app)

---

## The problem

Climate adaptation in cities increasingly depends on urban trees for cooling. Trees are effective natual coolers.
[see one of many LinkedIn posts on this](https://www.linkedin.com/posts/timschumacher_on-this-very-hot-day-a-small-reminder-on-activity-7464632411142746113-bS7q/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACQ0CcABNgihAWULriPVgpQEQYq9GKkJhgI).
But irrigation resources are limited, and not all trees are equal:

- Some species transpire (and therefore cool) more efficiently than others, relative to how much water they consume.
- Heat is not evenly distributed across a city - some streets run significantly hotter than others (urban heat island effect).
- Rain reduces irrigation need, but only some of it actually reaches the roots.

The dashboard below turns Frankfurt's open tree registry into a decision-support tool: for any tree, on any day, how much water does it need, and how much cooling value does that water buy?

---

## Demo

<img width="800" height="520" alt="ScreenRecording2026-06-17at15 32 02-ezgif com-video-to-gif-converter" src="https://github.com/user-attachments/assets/cd819cf0-327c-4c78-a427-2eec89741d10" />


---

## What this dasboard does

- **Maps** all approx. 130,000 matched **Frankfurt street trees**, clustered for performance
- **Pulls** tomorrow's **weather forecast** automatically every day
- **Calculates** **irrigation need** per tree (accounting for species, crown size, heat, humidity, and rainfall)
- **Calculates** the **cooling output** of that water (kWh equivalent)
- **Scores** each tree's structural **cooling efficiency** (height/crown ratio, species-adjusted)
- **Combines** **efficiency** with local urban **heat island intensity** into a **cooling priority score**
- Toggle a heat island overlay to see where the priority trees actually are

---

## Architecture

<img width="9414" height="5204" alt="Untitled" src="https://github.com/user-attachments/assets/b21bb3ef-fdcd-47f1-affc-93e3249de276" />


---

## Data pipeline

| Stage | Tool | What happens |
|---|---|---|
| Tree source data | Frankfurt Baumkataster (open data) | ~161k trees: species, crown diameter, height, planting year, location |
| Weather automation | Make.com + Open-Meteo | Daily scenario fetches tomorrow's forecast for Frankfurt, appends to Google Sheets |
| Heat island data | DWD HOSTRADA (May 2026, hottest month on record) | Hourly NetCDF raster, aggregated to ~575 grid cells (~1km² each) |
| Data enrichment | KNIME Analytics Platform | Coordinate conversion, spatial join (trees ↔ UHI grid), species lookup, formula calculations |
| Frontend | Lovable (React + Leaflet) | Map, clustering, weather card, water needs panel, area selection, heat overlay |

### KNIME workflow for data transformation

<img width="1255" height="425" alt="image" src="https://github.com/user-attachments/assets/a39b066d-4e45-4f5e-aec9-07387a590d4d" />

The workflow reads the raw tree CSV and the processed UHI grid, rounds coordinates to enable a spatial join, assigns species-specific water/shade coefficients via Rule Engine nodes, and calculates the final cooling efficiency and priority scores before exporting the enriched dataset.

Full technical breakdown: [`knime_workflows/README.md`](knime_workflows/README.md)

---

## The formulas

**Cooling Efficiency** (structural, weather-independent):
```
Cooling Efficiency = (Tree Height × Shade Factor) ÷ (Crown Diameter × Species Water Factor)
```
A unitless score — height and crown diameter cancel their units. Capped at 5.0, with guards for zero/missing crown or height.

**Irrigation Need** (daily, weather-dependent):
```
Crown Area = π × (Crown Diameter ÷ 2)²
Base Water Need = Crown Area × 3.5 × Species Water Factor (Kc)
Heat Adjustment = max(0, (Max Temp °C − 15) ÷ 10)
Total Water Consumed = Base Water Need × (1 + Heat Adjustment) × (1 − Humidity % ÷ 200)
Rain Offset = Precipitation (mm) × Crown Area
Irrigation Needed = max(0, Total Water Consumed − Rain Offset)
```

**Cooling Output:**
```
Cooling Output (kWh) = Total Water Consumed (L) × 0.7
```
*(1 litre of transpired water ≈ 0.7 kWh of cooling — based on the latent heat of vaporisation.)*

**Cooling Priority** (the key decision metric):
```
Cooling Priority = Cooling Efficiency × Local UHI Intensity
```
High value = an efficient cooler sitting in an already-hot zone → top watering priority.

### Species factors

Water (Kc) and shade coefficients are anchored to [WUCOLS](https://wucols.ucdavis.edu/) water-use bands and the [FAO-56](https://www.fao.org/4/x0490e/x0490e0b.htm) crop coefficient methodology, adapted for 10 common Frankfurt street tree genera (Platanus, Tilia, Acer, Robinia, Carpinus, Quercus, Fraxinus, Aesculus, Prunus, Fagus). Shade factors are directional estimates based on canopy density, not measured values.

---



## Tech stack

- **Data store:** Google Sheets
- **Automation:** Make.com (daily weather fetch via Open-Meteo)
- **ETL / enrichment:** KNIME Analytics Platform
- **Frontend:** Lovable (React, Leaflet, marker clustering, Leaflet.draw)
- **Heat island source:** DWD HOSTRADA via Python (xarray, netCDF4, pandas)

---

## Known limitations

- ~32k of 161k trees (≈20%) fall outside the UHI grid coverage and are excluded from the enriched dataset.
- Shade factors are directional estimates, not measured canopy density values — labelled as such.
- UHI intensity reflects May 2026 conditions (a record-hot month) and is static; it does not update daily.
- One data outlier (a tree with near-zero crown diameter) is capped rather than removed; negligible effect on the 1.52 efficiency threshold, which is based on the dataset median.

---

## Author

Built as a hands-on portfolio project to demonstrate end-to-end data pipeline design — from open data ingestion through ETL, automation, and interactive visualisation — applied to a real urban climate adaptation question.
