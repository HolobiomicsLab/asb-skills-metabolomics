---
name: mass-spectrometry-qc-tabulation
description: 'Use when vendor-agnostic QC of LC-HRMS acquisitions is needed: computing
  per-analyte retention time RSD, peak areas, or extracted-ion intensities from SST
  mixture mzML files to assess instrument performance before data collection.'
when_to_use_negative:
- Input data are already a feature/peak table rather than mzML files — MassQLab queries
  raw scan data directly
- Evaluation of MS/MS product ion consistency is required — the authors report DDA's
  stochastic MS/MS triggering is problematic for SST consistency assessment and MS/MS
  evaluation was not warranted in their use case
- Intensity and peak-shape assessment is required — these were not evaluated in the
  illustrative SST assessment
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MassQLab
  role: Python/Jupyter implementation of MassQL that applies a series of queries to
    a directory of mzML files and tabulates/visualizes results as images and csv files
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to express per-analyte MS1/MS2 extracted-ion queries with
    ppm tolerances and RT windows
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw files to mzML format (default settings) prior to MassQL
    querying
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Runtime for MassQLab (developed using Python 3.9); MassQL queries can also
    be run directly via the massql Python API
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:dc5c997a1a28dd2cb107f7c9743173b08c29f389fb6682e2cf14f4264bc99d17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mass-spectrometry-qc-tabulation

## Summary

Apply named MassQL queries to a directory of mzML files via MassQLab to tabulate and visualize system suitability metrics (e.g., retention time RSD of deuterated acylcarnitine SST analytes) across instruments and operators.

## When to use

Use when vendor-agnostic QC of LC-HRMS acquisitions is needed: computing per-analyte retention time RSD, peak areas, or extracted-ion intensities from SST mixture mzML files to assess instrument performance before data collection.

## When NOT to use

- Input data are already a feature/peak table rather than mzML files — MassQLab queries raw scan data directly
- Evaluation of MS/MS product ion consistency is required — the authors report DDA's stochastic MS/MS triggering is problematic for SST consistency assessment and MS/MS evaluation was not warranted in their use case
- Intensity and peak-shape assessment is required — these were not evaluated in the illustrative SST assessment

## Inputs

- Vendor-agnostic mass spectrometry data files in mzML format (converted from .raw via MSConvert)
- Query file (JSON, CSV, or XLSX) with 'name' and 'query' fields defining MassQL queries per analyte
- massqlab_config.json specifying data_directory, queryfile, and optional output_directory/analysis settings
- SST mixture mzML files (e.g., MassIVE MSV000098825, deuterated acylcarnitine standards at 1 ug/mL)

## Outputs

- ms1_raw_df.csv (raw query results + metadata) and ms1_analysis_df.csv (peak fitting analysis)
- ms1_RT_analysis_df.csv (retention time statistics vs. query RT)
- Summary plots (ms1_traces.pdf, ms1_summary_areas.pdf, ms1_consolidated_traces.png)
- ms2_raw_df.csv and ms2_analysis_df.csv when MS2 queries are applied
- Per-analyte retention time RSD statistics across acquisitions

## How to apply

Convert vendor .raw files to mzML with MSConvert (default settings) and download SST data (e.g., MassIVE MSV000098825). Define a query file (JSON/CSV/XLSX) with a unique 'name' and MassQL 'query' per analyte, e.g., MS1 extracted-ion queries like 'QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2' with RT windows. Configure massqlab_config.json with data_directory and queryfile, then run MassQLab_console.py, which applies each query to every mzML file, performs Gaussian peak fitting, and saves results as images and csv files (ms1_raw_df.csv, ms1_analysis_df.csv, ms1_RT_analysis_df.csv). Extract each analyte's detected retention time per acquisition and compute per-analyte RSD across acquisitions. Validate by confirming the computed RSD range (e.g., 0.5%-2.3% over ~6 months, two platforms, five operators in the published SST assessment) and that each peak meets detection criteria to be considered valid.

## Related tools

- **MassQLab** (Python/Jupyter implementation of MassQL that applies a series of queries to a directory of mzML files and tabulates/visualizes results as images and csv files) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to express per-analyte MS1/MS2 extracted-ion queries with ppm tolerances and RT windows) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw files to mzML format (default settings) prior to MassQL querying) — https://proteowizard.sourceforge.io/
- **Python** (Runtime for MassQLab (developed using Python 3.9); MassQL queries can also be run directly via the massql Python API)

## Examples

```
py src/MassQLab_console.py  # with massqlab_config.json: data_directory=SST mzML dir, queryfile=MassQL_Queries.json
```

## Evaluation signals

- Per-analyte retention time RSD falls within the expected range (published benchmark: 0.5%-2.3% across ~6 months, two instrument platforms, five operators; Table S2)
- Each detected peak meets MassQLab's detection criteria to be considered valid in downstream analysis
- Expected output files exist: ms1_raw_df.csv, ms1_analysis_df.csv, ms1_RT_analysis_df.csv, and summary PDFs in MassQLab_Output organized by timestamp
- Gaussian fits in ms1_traces.pdf align with the extracted-ion chromatogram peaks at the query-defined RT windows (e.g., RTMIN=1.0 AND RTMAX=1.2 for m/z 207.1418)
- Each query in the queryfile has a unique name per MS1/MS2 type and short names that do not break output paths

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in the authors' evaluation
- MassQLab's convert_raw raw-to-mzML feature is experimental; MSConvert is recommended instead
- MassQLab currently processes all files in the data_directory, and MassQLab performance/functions are still to be refined/expanded
- MassQL is tested in Python 3.9; other versions may not work

## Evidence

- [readme] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [results] Retention time RSD result: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [methods] File conversion prerequisite: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [readme] Example MS1 query: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [discussion] DDA MS/MS limitation: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
