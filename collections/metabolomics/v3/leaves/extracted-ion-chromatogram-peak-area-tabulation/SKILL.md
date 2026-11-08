---
name: extracted-ion-chromatogram-peak-area-tabulation
description: Use when SST mzML data (e.g., six deuterated acylcarnitine standards
  on UHPLC-HRMS) need EIC peak areas tabulated per analyte across acquisition batches
  to detect retention-time or method anomalies.
when_to_use_negative:
- Intensity and peak shape evaluation is required but not part of this skill's illustrative
  SST scope (the article explicitly notes 'the intensity and peak shape were not evaluated').
- The input data cannot be converted to mzML — vendor-specific file formats must first
  be converted to an open-source MS file format before MassQL use.
- 'A goal is assessing MS/MS consistency for SST generally: the authors state the
  stochastic nature of DDA MS/MS triggering is problematic and that SST MS/MS evaluation
  is not warranted in their use case.'
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassQLab
  role: Python-based Jupyter notebook/console implementation that applies named MassQL
    queries to a directory of mzML files, fits peaks, and tabulates results as images
    and csv files
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language used to express MS1/MS2 extracted-ion, precursor, product-ion,
    and neutral-loss filter criteria for each SST analyte
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw files to mzML format with default settings before MassQL
    querying
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:7522cee279d3011d5c377dac672b1f42d56cb7a0b1e8129aacea55185ea07a67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# extracted-ion-chromatogram-peak-area-tabulation

## Summary

Applies named MassQL queries (MS1 extracted-ion filters and MS2 precursor/product/neutral-loss criteria) to a directory of mzML files via MassQLab, calculates chromatographic peak areas, and tabulates results as csv files with graphical PDF outputs for system suitability testing (SST).

## When to use

Use when SST mzML data (e.g., six deuterated acylcarnitine standards on UHPLC-HRMS) need EIC peak areas tabulated per analyte across acquisition batches to detect retention-time or method anomalies.

## When NOT to use

- Intensity and peak shape evaluation is required but not part of this skill's illustrative SST scope (the article explicitly notes 'the intensity and peak shape were not evaluated').
- The input data cannot be converted to mzML — vendor-specific file formats must first be converted to an open-source MS file format before MassQL use.
- A goal is assessing MS/MS consistency for SST generally: the authors state the stochastic nature of DDA MS/MS triggering is problematic and that SST MS/MS evaluation is not warranted in their use case.

## Inputs

- Directory of mzML-format mass spectrometry files (SST acquisitions of six deuterated acylcarnitine standards)
- Query file (JSON, CSV, or XLSX) with name and query fields defining MassQL queries
- massqlab_config. with data_directory, queryfile, output_directory, analysis, cache_setting, convert_raw, msconvertexe fields

## Outputs

- ms1_raw_df.csv (raw query results + metadata)
- ms1_analysis_df.csv (peak fitting analysis) and ms1_RT_analysis_df.csv (RT stats vs. query RT)
- ms1_traces.pdf, ms1_summary_traces.pdf, ms1_summary_areas.pdf and consolidated PNG of Gaussian fits
- ms2_raw_df.csv, ms2_analysis_df.csv and ms2_plots.pdf / ms2_summary_plots.pdf / ms2_cluster_plots.pdf
- Results organized by timestamp under MassQLab_Output in the output (default: data) directory

## How to apply

Populate massqlab_config. with data_directory pointing to the mzML files (converted from vendor .raw with MSConvert default settings if needed) and queryfile pointing to MassQL_Queries./csv/xlsx containing named queries. Compose MassQL queries per analyte, e.g., QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2 for acetyl-L-carnitine-d3, and MS2 checks such as MS2PREC=429.3765:TOLERANCEPPM=2.5 with FILTER MS2PROD=85.0281:TOLERANCEPPM=10 or WHERE MS2NL=176.0321 AND MS2PROD=85.02915. Run the full pipeline (py src\MassQLab_console.py or notebooks/MassQLab_console.ipynb), which applies each named query to each mzML file and, with analysis=true, performs Gaussian peak fitting to compute extracted chromatographic peak areas. Results are written to MassQLab_Output inside the output directory (data_directory by default), organized by timestamp, as ms1_raw_df.csv, ms1_analysis_df.csv, ms1_RT_analysis_df.csv plus PDFs/PNG of traces and areas; MS2 outputs include ms2_raw_df.csv and clustered plot PDFs. Validate by confirming each detected SST peak meets the detection criteria and that retention-time RSD for non-anomalous batches falls in the reported 0.5%-2.3% range across SST analytes.

## Related tools

- **MassQLab** (Python-based Jupyter notebook/console implementation that applies named MassQL queries to a directory of mzML files, fits peaks, and tabulates results as images and csv files) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language used to express MS1/MS2 extracted-ion, precursor, product-ion, and neutral-loss filter criteria for each SST analyte) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw files to mzML format with default settings before MassQL querying) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- Each detected SST peak meets the detection criteria to be considered valid in MassQLab downstream analysis
- Retention-time RSD across all SST analytes (non-anomalous batches) falls within the reported 0.5% to 2.3% range over ~6 months, two platforms, and five operators
- ms1_analysis_df.csv contains Gaussian peak-fitting-derived areas for every named query/file pair present in ms1_raw_df.csv
- Output directory contains the full expected file set (ms1_*.csv/pdf/png and ms2_*.csv/pdf) organized by timestamp under MassQLab_Output
- Known anomalies (column-lot retention-time shifts, BEH Amide wrong-method EIC patterns) are visually evident in the generated EIC/trace plots when historical SST data are analyzed

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment.
- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to total signal helped in their evaluation.
- SST MS/MS evaluation is not warranted in the authors' use case.
- MassQLab's convert_raw feature is experimental; recommended practice is to convert files using ProteoWizard msconvert before use.
- MassQLab performance and available functions are planned to be refined/expanded in future work.

## Evidence

- [intro] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [methods] QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [readme] ms1_analysis_df.csv: Peak fitting analysis: "ms1_analysis_df.csv: Peak fitting analysis"
