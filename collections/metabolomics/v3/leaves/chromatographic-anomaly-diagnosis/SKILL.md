---
name: chromatographic-anomaly-diagnosis
description: Use when longitudinal LC-MS/MS SST data (e.g., deuterated acylcarnitine
  standards in mzML) must be screened for retention time drift, column-lot shifts,
  or wrong chromatographic method use prior to experimental acquisition.
when_to_use_negative:
- Intensity and peak shape evaluation is required — the illustrative SST assessment
  explicitly did not evaluate intensity and peak shape
- The goal is MS/MS consistency assessment via DDA SST — the stochastic nature of
  DDA MS/MS triggering is problematic for assessing consistency, and the authors state
  SST MS/MS evaluation is not warranted in their use case
- Input data are not LC-MS mzML-queryable (e.g., already-processed feature tables);
  MassQL operates on raw mzML scan data, so feature-level inputs bypass this skill
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassQLab
  role: Python-based Jupyter/console implementation that applies series of named MassQL
    queries to a directory of mzML files, tabulates results, and generates EIC/peak-area
    plots for SST anomaly diagnosis
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language used to express MS1/MS2 filters (e.g., MS1MZ,
    MS2PREC, MS2PROD with TOLERANCEPPM and RT windows) executed by MassQLab
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: ProteoWizard tool used with default settings to convert vendor .raw files
    into open-source mzML format required before MassQL querying
  repo: https://proteowizard.sourceforge.io/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:6047752765418c8f695d16336e6a007d3f6bf175d7719d0e73bb93c908786256
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
yes_rate: 1.0
n_claims: 15
---

# chromatographic-anomaly-diagnosis

## Summary

A system suitability testing (SST) skill that uses MassQLab (built on MassQL) to run named MassQL queries over directories of LC-HRMS mzML files, tabulate retention times and EIC peak areas, and diagnose chromatographic anomalies such as column-lot-linked retention time shifts and columns run with the wrong method before full data collection.

## When to use

Use when longitudinal LC-MS/MS SST data (e.g., deuterated acylcarnitine standards in mzML) must be screened for retention time drift, column-lot shifts, or wrong chromatographic method use prior to experimental acquisition.

## When NOT to use

- Intensity and peak shape evaluation is required — the illustrative SST assessment explicitly did not evaluate intensity and peak shape
- The goal is MS/MS consistency assessment via DDA SST — the stochastic nature of DDA MS/MS triggering is problematic for assessing consistency, and the authors state SST MS/MS evaluation is not warranted in their use case
- Input data are not LC-MS mzML-queryable (e.g., already-processed feature tables); MassQL operates on raw mzML scan data, so feature-level inputs bypass this skill

## Inputs

- LC-MS/MS SST data files in mzML format (or vendor .raw convertible via MSConvert), e.g., from MassIVE accession MSV000098825
- MassQL query file (.xlsx, .csv, or .) with 'name' and 'query' fields per analyte, e.g., six deuterated acylcarnitine SST queries
- massqlab_config. specifying data_directory and queryfile (plus optional output_directory, analysis, cache_setting, convert_raw, msconvertexe)

## Outputs

- Tabulated query results exported as .xlsx/.csv (ms1_raw_df.csv, ms1_analysis_df.csv, ms1_RT_analysis_df.csv, ms2_raw_df.csv, ms2_analysis_df.csv)
- PDF/PNG EIC and peak-area plots (ms1_traces.pdf, ms1_summary_areas.pdf, ms2_plots.pdf, etc.) for visual anomaly inspection
- Diagnosis of chromatographic anomalies: column-lot retention time shifts and wrong-method column runs, flagged against the 0.5%–2.3% RT RSD baseline

## How to apply

Convert vendor .raw files to mzML with MSConvert default settings, then compose a query file (.xlsx/.csv/.) with named MassQL queries for each SST analyte — e.g., an MS1 EIC query 'QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2' for acetyl-L-carnitine-d3 and an MS2 query filtering MS2PREC=429.3765 at 2.5 ppm and MS2PROD=85.0281 at 10 ppm. Configure massqlab_config. with data_directory and queryfile paths and run the console workflow (py src\MassQLab_console.py), which applies each named query to each mzML file, tabulates peak areas, and exports results as .xlsx/csv plus PDF EIC plots. Inspect retention times and EIC plots across the acquisition period (~6 months here) to flag batch-clustered retention time shifts attributable to a column lot change and EIC patterns inconsistent with the F5 method indicating a BEH Amide column run with the wrong chromatographic method. Validate by checking that non-anomalous SST batches reproduce the reported retention time RSD range of 0.5%–2.3% across all SST analytes and that the two anomalies are visually evident in the generated EIC plots.

## Related tools

- **MassQLab** (Python-based Jupyter/console implementation that applies series of named MassQL queries to a directory of mzML files, tabulates results, and generates EIC/peak-area plots for SST anomaly diagnosis) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language used to express MS1/MS2 filters (e.g., MS1MZ, MS2PREC, MS2PROD with TOLERANCEPPM and RT windows) executed by MassQLab) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (ProteoWizard tool used with default settings to convert vendor .raw files into open-source mzML format required before MassQL querying) — https://proteowizard.sourceforge.io/

## Examples

```
py src\MassQLab_console.py
```

## Evaluation signals

- Retention time RSD across all SST analytes in non-anomalous batches falls within the reported 0.5%–2.3% range over ~6 months, two instrument platforms, and five operators
- Column-lot change appears as batch-clustered retention time shifts in tabulated RTs and EIC plots across the acquisition period
- Wrong-method column run appears as EIC patterns inconsistent with the F5 method (e.g., anomalous acetyl-L-carnitine-d3 retention in the 1.0–1.2 min window at m/z 207.1418 ± 2.5 ppm)
- MassQL queries return valid detections meeting the specified tolerances (2.5 ppm MS1/precursor, 10 ppm MS2 product) with tabulated peak areas present in the exported results
- Output directory contains the expected structured artifacts (ms1_raw_df.csv, ms1_RT_analysis_df.csv, ms1_summary_traces.pdf, ms2_plots.pdf) organized by timestamp

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment, so the skill does not cover those performance metrics
- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to total signal helped in the authors' evaluation
- MassQLab currently processes all files in the configured data directory and convert_raw is experimental — files are recommended to be pre-converted with ProteoWizard msconvert
- MassQLab performance and available functions are still to be refined/expanded, and no software version number is provided

## Evidence

- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [methods] QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [readme] MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files.: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [results] Note, the intensity and peak shape were not evaluated.: "Note, the intensity and peak shape were not evaluated."
- [discussion] the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation.: "the stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, albeit normalization to the total signal helped in our evaluation."
