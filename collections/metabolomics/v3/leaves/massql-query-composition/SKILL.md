---
name: massql-query-composition
description: Use when you need to express targeted MS detection criteria (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5
  with RTMIN/RTMAX, MS2PREC, MS2PROD, MS2NL) as structured queries for SST or EIC/product-ion
  extraction from mzML data.
when_to_use_negative:
- Data are in vendor-specific .raw format and have not yet been converted to mzML
  — convert with MSConvert first, since vendor formats must be converted before MassQL
  use
- The analysis requires peak intensity and peak shape evaluation, which the paper's
  illustrative SST assessment explicitly did not evaluate
- You need day-to-day MS/MS product-ion consistency assessment via DDA, which the
  authors deemed unwarranted due to the stochastic nature of DDA triggering
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassQLab
  role: Python 3.9 Jupyter/console implementation that applies each named MassQL query
    to every mzML file in a directory, tabulates peak areas, and saves results as
    images and csv files
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language used to express the MS1/MS2 filters, tolerances,
    and RT windows executed against mzML data
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw files to mzML format with default settings, the prerequisite
    format for MassQL queries
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  - task_004
  - task_003
  - task_006
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:fa9d3c52fe6d0d8f1c30a3c2384b40b1664ba0c0438199a67b639e228055e248
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  - matplotlib
  - openpyxl
  - reportlab
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# massql-query-composition

## Summary

Compose named MassQL queries (MS1 extracted-ion, MS2 precursor/product-ion, and neutral-loss filters with ppm tolerances and RT windows) that MassQLab applies to a directory of mzML files to tabulate system suitability results for the six deuterated acylcarnitine SST analytes.

## When to use

Use when you need to express targeted MS detection criteria (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5 with RTMIN/RTMAX, MS2PREC, MS2PROD, MS2NL) as structured queries for SST or EIC/product-ion extraction from mzML data.

## When NOT to use

- Data are in vendor-specific .raw format and have not yet been converted to mzML — convert with MSConvert first, since vendor formats must be converted before MassQL use
- The analysis requires peak intensity and peak shape evaluation, which the paper's illustrative SST assessment explicitly did not evaluate
- You need day-to-day MS/MS product-ion consistency assessment via DDA, which the authors deemed unwarranted due to the stochastic nature of DDA triggering

## Inputs

- mzML-format mass spectrometry data files (converted from vendor .raw via MSConvert with default settings)
- Query file (JSON, CSV, or XLSX) with 'name' and 'query' fields per query, e.g., MassQL_Queries.
- massqlab_config. specifying data_directory and queryfile paths

## Outputs

- Tabulated query results (ms1_raw_df.csv, ms1_analysis_df.csv, ms2_raw_df.csv, ms2_analysis_df.csv) with peak areas and RT statistics
- Graphical outputs (ms1_traces.pdf, ms1_summary_areas.pdf, ms2_plots.pdf, ms2_cluster_plots.pdf) per named query/file
- Consolidated trace images (ms1_consolidated_traces.png)

## How to apply

Write one named query per analyte/check into a query file (JSON, CSV, or XLSX with 'name' and 'query' fields, e.g., MassQL_Queries.), following the pattern QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2 for extracted-ion chromatograms, or QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10 for product-ion checks. Use 2.5 ppm tolerance for precursor/extracted-ion masses and 10 ppm (or TOLERANCEPPM=10) for MS2 product ions; combine WHERE clauses with AND for co-occurring neutral losses and products (e.g., MS2NL=176.0321 AND MS2PROD=85.02915). MassQL supports arithmetic expressions such as MS2PROD=144+formula(CH2). Point massqlab_config.'s queryfile and data_directory at the query file and mzML folder, then run the MassQLab console pipeline so each named query is applied to every mzML file and results are tabulated. Keep query names unique per MS1/MS2 type and avoid very long names to prevent output issues.

## Related tools

- **MassQLab** (Python 3.9 Jupyter/console implementation that applies each named MassQL query to every mzML file in a directory, tabulates peak areas, and saves results as images and csv files) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language used to express the MS1/MS2 filters, tolerances, and RT windows executed against mzML data) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw files to mzML format with default settings, the prerequisite format for MassQL queries) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- Each detected SST peak must meet the detection criteria to be considered valid in downstream MassQLab analysis
- Per-analyte retention time RSD across the ~6-month, two-platform, five-operator acquisitions falls within the reported 0.5%–2.3% range (Table S2)
- Output files match the documented MassQLab schema: ms1_raw_df.csv, ms1_analysis_df.csv, ms2_raw_df.csv, and the corresponding PDF summaries organized by timestamp in MassQLab_Output
- Extracted m/z values honor the stated tolerances: 2.5 ppm for MS1MZ/MS2PREC queries and 10 ppm for MS2PROD filters
- Each query in the query file has a unique 'name' per MS1/MS2 type and executes without output issues (names not excessively long)

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- MassQL currently tests on Python 3.9; other versions are uncertain
- MassQLab's convert_raw feature is experimental; MSConvert is recommended for raw-to-mzML conversion
- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, though normalization to total signal helped

## Evidence

- [intro] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [methods] QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [readme] QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10: "QUERY scaninfo(MS2DATA) WHERE MS2PREC=429.3765:TOLERANCEPPM=2.5 AND RTMIN=9.0 AND RTMAX=9.5 FILTER MS2PROD=85.0281:TOLERANCEPPM=10"
- [intro] One prerequisite is that vendor-specific file formats must be converted into an open-source MS file format (i.e., .mzML).: "One prerequisite is that vendor‐specific file formats must be converted into an open‐source MS file format (i.e., .mzML)."
- [readme] Each query in the file must include the following: name: A unique name for each query (per MS1/MS2 type). Avoid very long names to prevent output issues.: "name: A unique name for each query (per MS1/MS2 type). Avoid very long names to prevent output issues."
