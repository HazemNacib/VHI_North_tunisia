# VHI_North_tunisia

A Jupyter notebook that computes drought indices (**VCI**, **TCI**, **VHI**) for
northern Tunisia from **MODIS satellite data (2001–2026)**, using **Google Earth
Engine**. It produces maps, time-series charts, and exportable NetCDF files.

---

## What it does

The notebook pulls two MODIS products for northern Tunisia (Bizerte, Ariana,
Tunis, Ben Arous, Manouba, Beja, Jendouba, Le Kef, Nabeul, Siliana, Zaghouan):

- **MOD13A2** — 16-day Vegetation Index (NDVI)
- **MOD11A2** — 8-day Land Surface Temperature (LST)

From these it derives three standard drought indicators:

| Index | Full name | Meaning |
|-------|-----------|---------|
| **VCI** | Vegetation Condition Index | How vegetation greenness compares to its historical range |
| **TCI** | Temperature Condition Index | How land surface temperature compares to its historical range |
| **VHI** | Vegetation Health Index | Combined score (50% VCI + 50% TCI) — the overall drought signal |

All three are scored 0–100, where lower = more severe drought.

---

## Requirements

1. **A Google Earth Engine (GEE) account.**
   Earth Engine is free for non-commercial/research use, but you must
   register and have it approved before the notebook will run:
   👉 https://earthengine.google.com/signup/

2. **A Google Cloud project linked to Earth Engine.**
   Once your GEE account is active, create (or pick) a Google Cloud project
   and register it for Earth Engine use at:
   👉 https://console.cloud.google.com/

3. **Python packages** (the notebook installs most of these itself in
   Cell 10, but you'll also need `earthengine-api` available in your
   Jupyter environment):
   - `earthengine-api` (`ee`)
   - `xee`, `geemap`, `rioxarray`
   - `xarray`, `matplotlib`, `pandas`, `seaborn`, `shapely`

---

## Setup — before you run it

### 1. Put your Earth Engine project ID in Cell 1

Open the notebook's **Cell 1 — Authenticate & Initialise Earth Engine** and
replace the placeholder project name with your own:

```python
import ee
ee.Authenticate()

project = 'YOUR-GOOGLE-EARTH-ENGINE-PROJECT-ID'   # <-- put your project ID here
ee.Initialize(project=project, opt_url='https://earthengine-highvolume.googleapis.com')
```

Your project ID is the short name shown in the Google Cloud Console (e.g.
`my-drought-project-123456`) — **not** your email address or account name.
This is the one line you must edit for the notebook to work with your
account; everything else runs as-is.

### 2. Authenticate

The first time you run Cell 1, `ee.Authenticate()` will open a browser
window (or print a link) asking you to log in with the Google account tied
to your Earth Engine project, then paste back a verification code. You only
need to do this once per machine — after that, Earth Engine remembers you.

### 3. Run the cells in order, top to bottom

The notebook is built so each cell depends on variables created in the
ones before it. Skipping cells or running them out of order will cause
`NameError`s.

---

## Notebook structure

| Cell | What it does |
|------|---------------|
| 1 | Authenticate & initialise Earth Engine — **edit your project ID here** |
| 2 | Study parameters — region of interest (ROI) and date range |
| 3 | Scaling & conversion functions (raw MODIS values → NDVI / °C) |
| 4 | Cloud-masking functions (discard low-quality pixels) |
| 5 | Monthly weighted-average function (turns 8/16-day composites into months) |
| 6 | Historical min/max function (baseline for each calendar month) |
| 7 | VCI · TCI · VHI index functions |
| 8 | Hydrological-year aggregation (Sep → Aug seasonal averages) |
| 9 | Runs the full pipeline end-to-end |
| 10 | Map production — installs packages, loads data as a gridded array, plots maps |
| 11 | Time-series charts (NDVI, LST, VCI, TCI, VHI trends over time) |
| 12 | Export — saves everything to NetCDF files and zips them up |

---

## Outputs

Running the export cell (Cell 12) creates:

```
/content/data/processed/
├── monthly/                              # monthly NDVI, LST, VCI, TCI, VHI (.nc)
├── yearly/                               # hydrological-year averages (.nc)
├── drought_indices_monthly.zip
└── drought_indices_hydrological_year.zip
```

`/content/...` is a Google Colab path. If you're running locally in
Jupyter, change `output_dir` in the export cell to a folder on your own
machine.

---

## Notes

- **Study region** and **date range** can be changed in Cell 2 (`roi` list
  and `start_time` / `end_time`).
- The **map-production cell (10)** must run before the **export cell (12)**,
  since the export step reuses grid/geometry variables built there.
- Google Earth Engine has usage quotas. If you hit `EEException: quota
  exceeded` errors, wait a bit or check your quota in the Cloud Console.
