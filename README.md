# Wellytics Analytics

A Jupyter notebook for generating patient activity analytics reports from the [Wellytics](https://wellytics.health) platform. It fetches data from the Wellytics API and exports a structured Excel report across three sheets.

---

## Features

- Fetches patient records filtered by organisation and date range
- Tracks uploaded clinical files per patient (excluding notes, transcriptions, and auto-generated content)
- Detects drug detection usage (transcription AI on/off) per patient
- Resolves assigned doctor names per patient
- Identifies AI CoPilot prompts used per patient
- Flags whether a clinical note was saved
- Exports a 3-sheet Excel report:
  - **Patient-wise Details** — one row per patient
  - **Summary by Doctor** — aggregated metrics per doctor
  - **Overall Summary** — organisation-level KPIs

---

## Prerequisites

- Python 3.8+
- Jupyter Notebook or JupyterLab
- The following packages (auto-installed by Cell 2):
  - `requests`
  - `pandas`
  - `openpyxl`

---

## Setup

1. **Clone or download** this repository and open `wellytics_analytics.ipynb` in Jupyter.

2. **Edit Cell 1 — CONFIG** with your values:

   ```python
   ENV       = "PROD"          # DEV | STAGE | DEMO | PROD
   ORG_ID    = "your-org-uuid" # Organisation UUID from Wellytics
   DATE_FROM = "22/02/26"      # DD/MM/YY or DD/MM/YYYY — leave blank for all time
   DATE_TO   = "22/03/26"      # DD/MM/YY or DD/MM/YYYY — leave blank for all time
   ```

3. **Run All Cells** (`Kernel → Restart & Run All`).

---

## Configuration

| Variable    | Description                                      | Example                                |
|-------------|--------------------------------------------------|----------------------------------------|
| `ENV`       | Target environment                               | `DEV`, `STAGE`, `DEMO`, `PROD`         |
| `ORG_ID`    | Organisation UUID from the Wellytics platform    | `360dd06e-ce7f-4d93-ba44-d9d63189f1d5` |
| `DATE_FROM` | Start date (inclusive). Leave blank for all time | `22/02/26`                             |
| `DATE_TO`   | End date (inclusive). Leave blank for all time   | `22/03/26`                             |

---

## API Environments

| ENV           | API Base URL                                          |
|---------------|-------------------------------------------------------|
| `DEV`         | `https://api-dev.platform.wellytics.health/auth/api`  |
| `STAGE`/`DEMO`| `https://api-stage.platform.wellytics.health/auth/api`|
| `PROD`        | `https://api.platform.wellytics.health/auth/api`      |

---

## Output

A file named `analytics_{ENV}_{ORG_ID[:8]}_{date_tag}.xlsx` is saved in the current working directory.

**Sheet 1 — Patient-wise Details**

| Column          | Description                                           |
|-----------------|-------------------------------------------------------|
| Date            | Patient record creation date                          |
| Doctor          | Assigned doctor name                                  |
| Patient         | Patient name                                          |
| OP / IP         | Outpatient or Inpatient                               |
| Files Uploaded  | Number of clinical files uploaded                     |
| Drug Detection  | Transcription AI status (`On` / `Off`) or blank       |
| Prompt Used     | CoPilot prompts run for this patient                  |
| Note Saved      | Whether a clinical note was saved (`Yes` / `No`)      |

**Sheet 2 — Summary by Doctor**

Aggregated count of patients, OP/IP split, files uploaded, notes saved, and prompts used — grouped by doctor.

**Sheet 3 — Overall Summary**

Organisation-level KPIs: total patients, file uploads, transcription usage, notes saved, and prompt usage for the selected period.

---

## Notebook Structure

| Cell   | Description                                              |
|--------|----------------------------------------------------------|
| Cell 1 | **CONFIG** — set `ENV`, `ORG_ID`, `DATE_FROM`, `DATE_TO` |
| Cell 2 | ENV setup, package install, authentication               |
| Cell 3 | Helper functions (date parsing, pagination, type resolution) |
| Cell 4 | Data fetching (patients, prompt keys, files, doctors)    |
| Cell 5 | ADA Discovery setup (transcription & prompt APIs)        |
| Cell 6 | Prompt usage fetch per patient                           |
| Cell 7 | Transcription / drug detection results                   |
| Cell 8 | Patient-wise detail table construction & display         |
| Cell 9 | Overall summary & doctor summary display                 |
| Cell 10| Excel export                                             |

---

## Notes

- The notebook auto-excludes non-clinical file types (`TEMP_PATIENT`, `NOTE`, `TRANSCRIPTION`, etc.) from file upload counts. Known clinical types such as `CLINICAL_NOTES`, `BLOOD_TEST`, `IMAGING`, and others are always counted.
- Doctor names are resolved from the patient detail endpoint with a fallback to a bulk user lookup.
- If both `DATE_FROM` and `DATE_TO` are left blank, all records for the organisation are fetched.
