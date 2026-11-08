---
name: lc-ms-system-suitability-testing
description: Use when LC-HRMS acquisitions (mzML of a spiked SST standard mixture)
  must be checked for system suitability — retention-time stability, EIC peak areas,
  or MS/MS product-ion/neutral-loss behavior — before collecting study data.
when_to_use_negative:
- The objective is evaluation of signal intensity or peak shape — the illustrative
  SST assessment explicitly did not evaluate intensity and peak shape
- Assessment depends on consistent DDA MS/MS triggering — the stochastic nature of
  MS/MS triggering under DDA is problematic for consistency assessment via SST
- Data cannot be provided in mzML form and cannot be converted (convert_raw is experimental;
  ProteoWizard msconvert conversion is recommended instead)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MassQLab
  role: Python-based Jupyter notebook/console implementation that applies a series
    of MassQL queries to a directory of mzML files and tabulates and visualizes the
    SST results
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Query language (e.g., MS1MZ/MS2PREC/MS2PROD/MS2NL filters with TOLERANCEPPM
    and RT windows) used to define per-analyte SST queries
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw SST acquisition files to mzML with default settings as
    a prerequisite for MassQL
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Language in which MassQL/MassQLab are implemented (Python 3.9; matplotlib/openpyxl/reportlab
    for plots and tabulated exports)
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:60636c1ec9d91772aaea84d412594f63e7e199e80a48cd3ea3a9773e57cdd4d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - JupyterLab
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  - matplotlib
  - openpyxl
  - reportlab
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# lc-ms-system-suitability-testing

## Summary

A system suitability testing (SST) workflow that runs MassQL queries via MassQLab over mzML acquisitions of a six-deuterated-acylcarnitine SST mixture on UHPLC-HRMS (Vanquish Horizon-Orbitrap Fusion / Fusion Lumos), tabulating retention times, peak areas, and MS/MS product-ion metrics to detect analytical performance deficits before real data collection.

## When to use

Use when LC-HRMS acquisitions (mzML of a spiked SST standard mixture) must be checked for system suitability — retention-time stability, EIC peak areas, or MS/MS product-ion/neutral-loss behavior — before collecting study data.

## When NOT to use

- The objective is evaluation of signal intensity or peak shape — the illustrative SST assessment explicitly did not evaluate intensity and peak shape
- Assessment depends on consistent DDA MS/MS triggering — the stochastic nature of MS/MS triggering under DDA is problematic for consistency assessment via SST
- Data cannot be provided in mzML form and cannot be converted (convert_raw is experimental; ProteoWizard msconvert conversion is recommended instead)

## Inputs

- SST mixture mzML data files (vendor .raw converted via MSConvert default settings), e.g. from MassIVE accession MSV000098825
- MassQL query file (JSON, CSV, or XLSX) with a unique 'name' and 'query' string per deuterated acylcarnitine analyte (MS1 or MS2 queries)
- massqlab_config.json specifying data_directory, queryfile, and optional analysis/cache_setting

## Outputs

- ms1_raw_df.csv, ms1_analysis_df.csv, and ms1_RT_analysis_df.csv tabulating per-query-per-file peak metrics and RT statistics
- ms1_traces.pdf, ms1_summary_traces.pdf, ms1_summary_areas.pdf and consolidated EIC/Gaussian-fit PNG images
- ms2_raw_df.csv, ms2_analysis_df.csv, ms2_plots.pdf, ms2_summary_plots.pdf, ms2_cluster_plots.pdf for MS/MS product-ion and neutral-loss results
- All outputs organized by timestamp in a MassQLab_Output directory inside the output_directory (data_directory by default)

## How to apply

Prepare an SST mixture of six deuterated acylcarnitine standards at 1 μg/mL and acquire LC-MS/LC-MS/MS data in positive ionization mode (F5 analytical column, 2.1 × 100 mm, 100 Å, 2.6 μm) over the study period; convert vendor .raw files to mzML with MSConvert using default settings, since MassQL requires the open-source .mzML format. Write one named MassQL query per analyte in a query file (JSON/CSV/XLSX, each entry with a 'name' and 'query'), e.g. MS1 extracted-ion queries such as 'QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2' or MS2 queries filtering MS2PROD=85.0281:TOLERANCEPPM=10, then point massqlab_config.json's data_directory and queryfile at the mzML folder and query file and run the MassQLab console workflow (py src\MassQLab_console.py), which applies every query to every mzML file and tabulates plus visualizes results in a timestamped MassQLab_Output directory. Extract per-analyte retention times (or peak areas, MS/MS product-ion intensities) from ms1_raw_df.csv/ms1_analysis_df.csv and compute statistics — e.g., per-analyte retention-time RSD across all acquisitions spanning ~6 months, two instrument platforms, and five operators; accept SST if each analyte's RT RSD falls within the reported 0.5%–2.3% range (Table S2). Decision point: deficits found before real data collection are flagged and corrected, which is the entire rationale for running SST prior to acquisition.

## Related tools

- **MassQLab** (Python-based Jupyter notebook/console implementation that applies a series of MassQL queries to a directory of mzML files and tabulates and visualizes the SST results) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Query language (e.g., MS1MZ/MS2PREC/MS2PROD/MS2NL filters with TOLERANCEPPM and RT windows) used to define per-analyte SST queries) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw SST acquisition files to mzML with default settings as a prerequisite for MassQL) — https://proteowizard.sourceforge.io/
- **Python** (Language in which MassQL/MassQLab are implemented (Python 3.9; matplotlib/openpyxl/reportlab for plots and tabulated exports))

## Examples

```
py src\MassQLab_console.py  # after pointing massqlab_config.json data_directory at the SST mzML files and queryfile at MassQL_Queries.json
```

## Evaluation signals

- Per-analyte retention-time RSD across all SST acquisitions falls within the reported 0.5%–2.3% range (Table S2), i.e., min ≈ 0.5% and max ≈ 2.3%
- Each named query in the query file was applied to every mzML file, with results tabulated (ms1_raw_df.csv/ms1_analysis_df.csv) and plots compiled in a timestamped MassQLab_Output directory
- Every detected SST analyte peak meets MassQLab's detection criteria before being accepted as valid in downstream analysis
- MS/MS differential check: trimethylamine neutral-loss signal is strong for CID at NCE above 30 but near-undetectable for HCD
- MassQL query strings respect tolerance windows (e.g., TOLERANCEPPM=2.5 for MS1MZ=207.1418; TOLERANCEPPM=10 for MS2PROD=85.0281) and RT windows

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- The stochastic nature of MS/MS triggering using DDA is problematic for assessing consistency using SST, although normalization to total signal helped in the authors' evaluation
- MassQLab performance and available functions are still to be refined/expanded; the standalone GUI is in development and raw-file conversion (convert_raw) is experimental
- Reported SST reproducibility depends on uncontrolled variables such as different personnel, columns, and LC solvents, which the authors deemed acceptable

## Evidence

- [results] Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [methods] MSConvert mzML conversion prerequisite: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [methods] SST mixture preparation: "The SST mixture was prepared to a final concentration of 1 μg/mL via the addition of each deuterated standard into 100 mL volumetric glassware"
- [readme] MassQLab operating model: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format."
- [abstract] Motivation for SST skill: "We identified a need based on literature reports that stated only 2% of papers performed system suitability testing."
- [methods] MS1 query example with ppm tolerance: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
