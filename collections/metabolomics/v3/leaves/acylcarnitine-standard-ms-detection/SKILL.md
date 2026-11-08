---
name: acylcarnitine-standard-ms-detection
description: 'Use when assessing LC-HRMS system suitability with deuterated acylcarnitine
  standards: you have mzML (or .raw convertible) SST runs and need reproducible EIC
  peak areas and RT statistics via MassQL queries.'
when_to_use_negative:
- Data are not in mzML format and cannot be converted — MassQL requires vendor-agnostic
  open MS file formats
- The SST assessment requires intensity and peak-shape evaluation — the illustrative
  MassQLab workflow explicitly did not evaluate intensity or peak shape
- You need MS/MS reproducibility assessment via DDA — the stochastic nature of DDA
  MS/MS triggering is problematic for SST consistency evaluation in this use case
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassQLab
  role: Python/Jupyter implementation that applies the series of MassQL queries to
    the mzML directory and tabulates/visualizes peak results for the deuterated acylcarnitine
    SST standards
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language used to express MS1/MS2 m/z, ppm tolerance,
    RT window, precursor, product-ion, and neutral-loss criteria
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw SST files to mzML format with default settings before
    MassQL querying
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:2f5bceafd5215c3efe2d453403845bcf6680ddcc50ee2cd96e06a1956b99aecd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# acylcarnitine-standard-ms-detection

## Summary

Detect and quantify six deuterated acylcarnitine SST standards in vendor-agnostic mzML data by composing MassQL MS1 extracted-ion, MS2 precursor, product-ion, and neutral-loss queries, then tabulating peak areas and retention-time statistics via MassQLab.

## When to use

Use when assessing LC-HRMS system suitability with deuterated acylcarnitine standards: you have mzML (or .raw convertible) SST runs and need reproducible EIC peak areas and RT statistics via MassQL queries.

## When NOT to use

- Data are not in mzML format and cannot be converted — MassQL requires vendor-agnostic open MS file formats
- The SST assessment requires intensity and peak-shape evaluation — the illustrative MassQLab workflow explicitly did not evaluate intensity or peak shape
- You need MS/MS reproducibility assessment via DDA — the stochastic nature of DDA MS/MS triggering is problematic for SST consistency evaluation in this use case

## Inputs

- SST mzML files (vendor .raw converted via MSConvert default settings) in a data directory
- MassQL query file (JSON, CSV, or XLSX) with name and query fields per acylcarnitine standard
- massqlab_config. specifying data_directory, queryfile, and optional output_directory/analysis settings

## Outputs

- ms1_raw_df.csv (raw MS1 query results), ms1_analysis_df.csv (Gaussian peak fitting), ms1_RT_analysis_df.csv (RT statistics vs query RT)
- ms2_raw_df.csv (raw MS2 query results) and ms2_analysis_df.csv (peak picking analysis)
- PDF/PNG reports: ms1_traces.pdf, ms1_summary_areas.pdf, ms1_consolidated_traces.png, ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf

## How to apply

Convert vendor .raw files to open mzML with MSConvert (default settings), since MassQL requires vendor-agnostic mzML input. Populate massqlab_config. with data_directory (SST mzML files) and queryfile (e.g. MassQL_Queries. with name/query pairs), then compose per-standard queries: MS1 extracted-ion filters such as QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2 for the common acylcarnitine fragment, MS2 precursor windows (e.g. MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5) with product-ion filters (MS2PROD=85.0281:TOLERANCEPPM=10) and neutral-loss criteria (MS2NL=176.0321 AND MS2PROD=85.02915). Run the MassQLab console pipeline (src/MassQLab_console.py or notebooks/MassQLab_console.ipynb), which applies each named query to every mzML file, Gaussian-fits MS1 chromatographic peaks, and tabulates peak areas. Judge correctness by checking each detected SST peak meets the detection criteria to be considered valid and that retention-time RSDs across SST analytes fall in the reported 0.5%–2.3% range.

## Related tools

- **MassQLab** (Python/Jupyter implementation that applies the series of MassQL queries to the mzML directory and tabulates/visualizes peak results for the deuterated acylcarnitine SST standards) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language used to express MS1/MS2 m/z, ppm tolerance, RT window, precursor, product-ion, and neutral-loss criteria) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw SST files to mzML format with default settings before MassQL querying) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- Each detected SST peak meets the MassQLab detection criteria to be considered valid
- Retention-time RSD across the six deuterated acylcarnitine analytes falls within the reported 0.5%–2.3% range
- ms1_analysis_df.csv contains Gaussian peak-area fits for every named MS1 query across each mzML file
- Query precision is confirmed by tolerance settings matching the article: 2.5 ppm for MS1MZ/MS2PREC, 10 ppm for MS2PROD, with RT windows such as 1.0–1.2 and 9.0–9.5 minutes

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- SST MS/MS evaluation is not warranted in the authors' use case; the stochastic nature of DDA MS/MS triggering is problematic for consistency assessment, albeit normalization to total signal helped
- MassQLab's convert_raw option is experimental; converting files with ProteoWizard msconvert beforehand is recommended
- Query names must be unique and not very long to prevent output issues

## Evidence

- [intro] MassQLab applies a series of MassQL queries to a directory of mzML data and saves results as images and csv files: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [methods] MS1 extracted-ion filter example: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [methods] MS2 precursor and product-ion filter examples: "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10"
- [results] Retention time RSD results: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [readme] mzML conversion prerequisite: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
