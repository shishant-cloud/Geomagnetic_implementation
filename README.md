# A Machine Learning-Based Framework for Geomagnetic Storm Detection and Prediction

**Project status:** Data acquisition and initial preprocessing implemented; machine-learning experiments are pending.

## Overview

This research project aims to develop a machine-learning framework for geomagnetic storm detection and prediction using multivariate geomagnetic time-series data.

The current implementation focuses on acquiring one-minute geomagnetic observations from the Kakioka (KAK) geomagnetic observatory, parsing daily IAGA-2002 `.min` files, combining observations into yearly datasets, performing quality-control checks, and visualizing data coverage and geomagnetic field variations.

These steps establish the initial data-processing pipeline required for subsequent time-series analysis and machine-learning experiments.

## Research Objectives

The broader research project aims to:

- Develop a reliable pipeline for processing multivariate geomagnetic time-series observations.
- Identify data gaps, missing measurements, duplicate timestamps, and irregular sampling intervals.
- Prepare clean, chronologically ordered datasets for exploratory data analysis.
- Establish a foundation for geomagnetic storm detection and forecasting.
- Evaluate machine-learning models after the dataset and prediction targets have been defined.

**Current scope:** The implemented work documented here covers data acquisition, parsing, quality control, and visualization. Storm classification and forecasting models have not yet been implemented or evaluated.

## Work Completed So Far

### 1. Project structure and data organization

A project directory was established in Google Drive with separate locations for raw data, processed datasets, downloader scripts, tables, and figures.

The main project directories include:

```text
Geomagnetic_Project/
├── data/
│   ├── raw/
│   │   └── KAK/
│   │       ├── 2018/
│   │       ├── 2019/
│   │       ├── 2020/
│   │       └── ...
│   └── processed/
│       └── KAK/
├── download_scripts/
│   └── KAK/
├── results/
│   ├── figures/
│   └── tables/
└── Geomagnetic.ipynb
```

The directory names describe the intended organization; the notebook does not establish that every processed-year directory contains a completed dataset.

### 2. Geomagnetic data acquisition

Year-specific Python downloader scripts were organized for the period 2018–2026:

- `KAK_18.py`
- `KAK_19.py`
- `KAK_20.py`
- `KAK_21.py`
- `KAK_22.py`
- `KAK_23.py`
- `KAK_24.py`
- `KAK_25.py`
- `KAK_26.py`

The notebook records successful downloads and file checks for 2018, 2019, and 2020. A later directory check reports 365 `.min` files for each year from 2018 through 2026.

**Data-coverage verification is still required:** the recorded count of 365 files for 2024 is one short of its 366 calendar days, and 2026 is not a complete calendar year. The current counts should therefore be treated as acquisition progress, not proof of complete coverage. The actual dates and contents of the downloaded files must be checked.

### 3. Data format and measurement variables

The inspected files use the IAGA-2002 format. Their headers identify the data source as the Japan Meteorological Agency and the observatory as Kakioka, with the IAGA station code `KAK`.

The notebook parses the following fields:

| Column | Description |
|---|---|
| `datetime` | Combined date and time of observation |
| `KAKX` | Geomagnetic X component |
| `KAKY` | Geomagnetic Y component |
| `KAKZ` | Geomagnetic Z component |
| `KAKG` | Additional geomagnetic field measurement provided in the source file |

The parser reads the measurement header, extracts the data rows, converts the four measurement columns to floating-point values, and combines the date and time into a pandas datetime column.

The resulting table contains five columns: one timestamp and four geomagnetic measurements.

### 4. Reusable parsing and quality-control functions

The notebook implements a reusable function, `parse_kak_min_file()`, for reading daily `.min` files. It locates the measurement header, extracts the date, time, and four field measurements, and returns a structured pandas DataFrame.

A separate function, `quality_check_kak()`, checks:

- Number of records.
- Missing timestamps and measurement values.
- Duplicate timestamps.
- Earliest and latest timestamps.
- One-minute sampling intervals.
- Non-one-minute intervals.

Separating parsing from quality control makes it easier to test the processing logic on individual files before applying it to larger datasets.

### 5. Processing and combining the 2020 dataset

The notebook processes the available 2020 daily files individually, collects the parsed DataFrames, and combines them into one chronologically sorted dataset.

The recorded processing result is:

| Metric | Result |
|---|---:|
| Daily files processed successfully | 365 |
| Failed files | 0 |
| Total records | 525,600 |
| Columns | 5 |
| First timestamp | 2020-01-01 00:00 |
| Last timestamp | 2020-12-30 23:59 |
| Duplicate timestamps | 0 |
| Missing measurement values | 0 |
| One-minute intervals between consecutive records | 525,599 |
| Non-one-minute intervals within the recorded series | 0 |

All 365 files were parsed successfully in the recorded run. The resulting dataset has no duplicate timestamps or missing values in the five parsed columns.

However, the dataset is **not a complete 2020 calendar-year dataset**. The final date is December 30, not December 31.

### 6. Missing-date detection

A further quality-control step compares the actual timestamps against an expected one-minute timestamp index for the entire calendar year.

Since 2020 was a leap year, a complete year should contain:

`366 × 1,440 = 527,040` minute-level timestamps.

The recorded comparison found:

| Metric | Result |
|---|---:|
| Expected timestamps | 527,040 |
| Actual timestamps | 525,600 |
| Missing timestamps | 1,440 |
| Missing calendar dates | 2020-12-31 |
| Duplicate timestamps | 0 |
| Missing values in `KAKX`, `KAKY`, `KAKZ`, `KAKG` | 0 each |

This identifies one entirely absent day: December 31, 2020.

The gap is caused by the absence of that day's raw file in the recorded dataset, rather than missing values inside the parsed records. The notebook correctly identifies the missing date and does not fill the gap through interpolation.

The missing day should remain documented until the source data can be retrieved or its unavailability verified.

### 7. Saved quality-control report

A CSV quality-control summary was generated for 2020:

`results/tables/KAK_QC_2020.csv`

The report records the expected and available days, missing date, expected and actual minute counts, missing timestamps, duplicate timestamps, and missing measurement counts.

This creates a record of the dataset's limitations and provides a starting point for automating quality checks across additional years.

### 8. Data visualization

The notebook creates and saves figures to `results/figures/`:

| Figure | Purpose |
|---|---|
| `KAK_data_acquisition_progress.png` | Visualizes the reported number of daily files acquired per year |
| `KAK_expected_vs_available_days.png` | Compares expected and reported daily-file counts |
| `KAK_2020_daily_data_coverage.png` | Illustrates the missing date in the 2020 dataset |
| `KAK_2020_components_first_week.png` | Plots all four geomagnetic measurements for January 1–7, 2020 |
| `KAK_2020_KAKX_first_week.png` | Plots the X component over the first week |
| `KAK_2020_KAKY_first_week.png` | Plots the Y component over the first week |
| `KAK_2020_KAKZ_first_week.png` | Plots the Z component over the first week |
| `KAK_2020_KAKG_first_week.png` | Plots the additional G measurement over the first week |

The figures are saved at 300 DPI. The notebook also includes code to package the generated figures into a ZIP archive.

The acquisition and availability figures must be interpreted cautiously until all yearly file counts have been validated against actual calendar dates.

## Technologies Used

- **Python** — data-processing implementation.
- **Pandas** — tabular data manipulation, datetime handling, sorting, and CSV export.
- **NumPy** — imported for numerical processing.
- **Matplotlib** — visualization of data coverage and geomagnetic measurements.
- **Python standard library** — filesystem operations, regular expressions, file parsing, and directory management.
- **Google Colab and Google Drive** — notebook execution and project-file storage.

## Current Results and Limitations

The notebook demonstrates that the daily-file parser can process the available 2020 files and that the resulting records can be combined into a one-minute time series.

The main verified outcome is a 525,600-row dataset with four geomagnetic measurement columns and a timestamp column, together with quality-control checks that identify an absent calendar day.

The following work has **not yet been demonstrated** in the notebook:

- Recovery and validation of all missing daily observations across the acquisition period.
- A finalized, complete multiyear processed dataset.
- Integration of solar-wind or interplanetary magnetic-field measurements.
- Integration of the Dst geomagnetic activity index.
- A defined and validated geomagnetic storm classification target.
- Feature engineering for a trained predictive model.
- Training, validation, and testing of machine-learning models.
- Forecasting results, performance metrics, or comparison with baseline models.

The notebook's later cells also report that a search of the current Google Drive runtime could not find the raw `.min` files. The previously recorded outputs therefore do not guarantee that the data are currently accessible. The Drive paths and saved artifacts should be verified before rerunning the complete pipeline.

## Next Steps

1. **Verify raw-data coverage:** inspect the filenames and dates for every year, paying particular attention to 2024 and the partial year 2026.
2. **Restore and validate the working directories:** ensure the downloaded `.min` files and project folders are accessible in the current runtime.
3. **Complete the preprocessing pipeline:** process each validated year and generate a quality-control report for every year.
4. **Resolve or document missing observations:** retrieve available missing files where possible and preserve unresolved gaps rather than silently imputing them.
5. **Validate the merged dataset:** check timestamp uniqueness, chronological ordering, measurement validity, and expected sampling intervals.
6. **Expand the research dataset:** incorporate suitable solar-wind, interplanetary magnetic-field, and geomagnetic activity data if required by the final research design.
7. **Define detection and prediction tasks:** specify the storm-labeling criteria and forecasting horizon, supported by appropriate scientific literature.
8. **Establish baseline models:** develop time-aware training, validation, and test splits and compare baseline approaches before evaluating more advanced machine-learning methods.
9. **Evaluate and document results:** report suitable detection or forecasting metrics, limitations, and reproducible experiment settings.

## Research Status

At this stage, the project has established an initial geomagnetic data-processing workflow, implemented a reusable parser and quality-control checks, processed the available 2020 daily files, identified a missing calendar day, and generated preliminary data-coverage and measurement plots.

The next phase is to validate the multiyear dataset and build the machine-learning experiments needed to evaluate geomagnetic storm detection and prediction.

*This README describes the work recorded in the current notebook. It does not claim that a machine-learning model has been trained or that the complete research framework has already been validated.*
