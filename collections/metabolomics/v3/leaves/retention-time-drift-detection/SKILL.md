---
name: retention-time-drift-detection
description: Use when historical or incoming LC-MS system suitability mzML files must
  be screened for batch-clustered retention time shifts or EIC patterns inconsistent
  with the intended chromatographic method (e.g., an F5 method run on a BEH Amide
  column) prior to data collection.
when_to_use_negative:
- Intensity and peak shape evaluation is needed — the illustrative SST assessment
  explicitly did not evaluate intensity or peak shape
- The goal is quantitative metabolomics feature detection on biological samples rather
  than system suitability screening of SST standards
- Data cannot be converted to mzML — vendor-specific formats must first be converted
  to mzML before MassQL can be applied
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassQLab
  role: Python/Jupyter console or GUI that applies named MassQL queries to a directory
    of mzML files, calculates peak areas, tabulates results, and generates EIC/area
    PDF plots for SST review
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language (SQL-inspired) used to express m/z, ppm tolerance,
    RT window, precursor, and product-ion filters for the SST analytes
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: ProteoWizard tool used to convert vendor .raw files to mzML with default settings
    before MassQL querying
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:3c004ad2b45656172c4f634c4bec530461a2c5061e720c662c2dbb69465c48e0
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# retention-time-drift-detection

## Summary

A MassQLab/MassQL-based system suitability workflow that detects retention time drift (e.g., column-lot-linked shifts) and wrong-column/wrong-method anomalies in LC-MS runs by querying deuterated acylcarnitine SST analytes in mzML files before full experimental data collection.

## When to use

Use when historical or incoming LC-MS system suitability mzML files must be screened for batch-clustered retention time shifts or EIC patterns inconsistent with the intended chromatographic method (e.g., an F5 method run on a BEH Amide column) prior to data collection.

## When NOT to use

- Intensity and peak shape evaluation is needed — the illustrative SST assessment explicitly did not evaluate intensity or peak shape
- The goal is quantitative metabolomics feature detection on biological samples rather than system suitability screening of SST standards
- Data cannot be converted to mzML — vendor-specific formats must first be converted to mzML before MassQL can be applied

## Inputs

- Directory of LC-MS mzML files (historical SST acquisitions; convert vendor .raw via MSConvert default settings)
- MassQL query file (.xlsx, .csv, or .) with 'name' and 'query' fields per analyte
- massqlab_config. specifying data_directory, queryfile, and optional output_directory/analysis/cache_setting/convert_raw/msconvertexe

## Outputs

- ms1_raw_df.csv and ms1_RT_analysis_df.csv tabulating per-query, per-file retention times and RT statistics
- ms1_summary_traces.pdf / ms1_summary_areas.pdf with Gaussian-fitted EIC plots per query and per file
- ms2_raw_df.csv and ms2_summary_plots.pdf for MS2 product-ion checks
- Flagged anomalies: batch-clustered retention time shifts (column lot change) and wrong-method EIC patterns

## How to apply

Download the SST mzML files (e.g., MassIVE MSV000098825), converting vendor .raw files with MSConvert default settings if needed. Compose a query file (.xlsx/.csv/.) with named MassQL EIC queries for the six deuterated acylcarnitine SST analytes — e.g., 'QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2' for acetyl-L-carnitine-d3, plus an MS2 check 'QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10'. Configure massqlab_config. with the mzML data_directory and queryfile path, then run the console workflow, which applies each named query to each mzML file, tabulates results, and generates summary PDFs of peak areas and Gaussian-fitted EIC traces. Inspect retention times and EIC plots across the acquisition period to flag batch-clustered retention time shifts attributable to a column lot change and EIC patterns inconsistent with the F5 method (indicating a BEH Amide column run with the wrong method); correct these issues before initiating full experimental analysis.

## Related tools

- **MassQLab** (Python/Jupyter console or GUI that applies named MassQL queries to a directory of mzML files, calculates peak areas, tabulates results, and generates EIC/area PDF plots for SST review) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language (SQL-inspired) used to express m/z, ppm tolerance, RT window, precursor, and product-ion filters for the SST analytes) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (ProteoWizard tool used to convert vendor .raw files to mzML with default settings before MassQL querying) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py  # after configuring massqlab_config. with data_directory, queryfile, and MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2 queries
```

## Evaluation signals

- Retention time RSD of non-anomalous SST batches should reproduce the reported 0.5%–2.3% range across all six deuterated acylcarnitine SST analytes over ~6 months, two instrument platforms, and five operators
- ms1_raw_df.csv and ms1_RT_analysis_df.csv should contain one row per named query per mzML file with fitted RT values near the query RT windows (e.g., ~1.0–1.2 min for m/z 207.1418)
- The two known performance anomalies should be visually evident in the generated EIC PDF plots: batch-clustered RT shifts (column lot change) and EIC patterns inconsistent with the F5 method (BEH Amide wrong-method run)
- MS2 check for MS2PREC=429.3765 (±2.5 ppm, RT 9.0–9.5 min) should return scans containing the MS2PROD=85.0281 peak (±10 ppm)
- Output directory should contain the timestamped MassQLab_Output folder with all expected ms1_/ms2_ CSV and PDF artifacts

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- The stochastic nature of DDA MS/MS triggering is problematic for assessing consistency using SST, although normalization to total signal helped in the authors' evaluation
- SST MS/MS evaluation is not warranted in the authors' use case since product ions and relative intensities were already reproducible
- MassQLab's convert_raw feature is experimental — ProteoWizard msconvert is recommended for raw-to-mzML conversion instead
- MassQLab performance and available functions are still to be refined/expanded, and no software version number is provided

## Evidence

- [results] Using the SST strategy empowered by MassQLab, performance issues — including column-lot-linked retention time shifts and erroneous EICs from a BEH Amide column run with the wrong method — were successfully recognized and corrected prior to initiating full experimental analysis.: [evidence span withheld — anchor status: not_found]
- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%.: "yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [readme] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format"
- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings"
- [results] Note, the intensity and peak shape were not evaluated.: "Note, the intensity and peak shape were not evaluated"
