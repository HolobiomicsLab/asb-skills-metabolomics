---
name: retention-time-precision-statistics
description: 'Use when system suitability testing must report chromatographic reproducibility:
  retention times of SST standard compounds are available per acquisition file across
  multiple runs, platforms, or operators, and RSD summaries are needed.'
when_to_use_negative:
- Intensity or peak shape reproducibility is the target — the authors explicitly did
  not evaluate intensity and peak shape in this SST assessment
- Data are MS/MS product-ion spectra from DDA acquisitions — the stochastic nature
  of DDA MS/MS triggering is problematic for assessing consistency via SST
- Input data are vendor .raw files — they must first be converted to mzML (e.g., via
  MSConvert) before MassQL querying
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassQLab
  role: Python/Jupyter console workflow that applies MassQL queries to a directory
    of mzML files and tabulates retention time and peak metrics as csv/pdf outputs
  repo: https://github.com/JohnsonDylan/MassQLab
- name: MassQL
  role: Domain-specific query language used to extract MS1 extracted-ion retention
    times per analyte (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5 with RTMIN/RTMAX windows)
  repo: https://github.com/mwang87/MassQueryLanguage
- name: MSConvert
  role: Converts vendor .raw SST acquisition files to mzML with default settings before
    querying
  repo: https://proteowizard.sourceforge.io/
- name: Python
  role: Language used to compute per-analyte relative standard deviations from the
    MassQLab result tables (MassQLab developed using Python 3.9)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:a51fe655d4e9b467c4db8d035005b95e94dd9f6b3825f75d954bf0397dfe0932
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JohnsonDylan/MassQLab
  tools:
  - MSConvert
  - MassQL
  - MassQLab
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# retention-time-precision-statistics

## Summary

Quantifies retention time reproducibility of system suitability (SST) analytes — here six deuterated acylcarnitine standards on Vanquish Horizon-Orbitrap Fusion/Fusion Lumos UHPLC-HRMS — by computing per-analyte relative standard deviation (RSD) across acquisitions, files, and operators.

## When to use

Use when system suitability testing must report chromatographic reproducibility: retention times of SST standard compounds are available per acquisition file across multiple runs, platforms, or operators, and RSD summaries are needed.

## When NOT to use

- Intensity or peak shape reproducibility is the target — the authors explicitly did not evaluate intensity and peak shape in this SST assessment
- Data are MS/MS product-ion spectra from DDA acquisitions — the stochastic nature of DDA MS/MS triggering is problematic for assessing consistency via SST
- Input data are vendor .raw files — they must first be converted to mzML (e.g., via MSConvert) before MassQL querying

## Inputs

- mzML files of SST mixture acquisitions (MassIVE MSV000098825)
- MassQLab tabulated results (ms1_RT_analysis_df.csv, ms1_raw_df.csv) with per-file detected retention times per analyte
- Named MassQL query file (JSON/CSV/XLSX) defining per-analyte m/z and RT windows

## Outputs

- Per-analyte retention time RSD (%) across all acquisitions
- Minimum/maximum RSD envelope for the SST analyte set (expected ~0.5%–2.3%)
- Summary statistics comparable to Table S2

## How to apply

Extract the detected retention time of each SST analyte per acquisition file from MassQLab output tables (e.g., ms1_RT_analysis_df.csv / ms1_raw_df.csv), which are produced by applying named MassQL MS1 extracted-ion queries such as 'QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2' to mzML files. Pool acquisitions spanning the full assessment window (~6 months, two instrument platforms, five operators) and compute per-analyte RSD (standard deviation / mean × 100) of the detected retention times. Judge acceptability against the reported envelope of 0.5%–2.3% RSD (Table S2); the authors deemed this acceptable given uncontrolled variables such as different personnel, columns, and LC solvents. Note that intensity and peak shape were intentionally not evaluated in this assessment.

## Related tools

- **MassQLab** (Python/Jupyter console workflow that applies MassQL queries to a directory of mzML files and tabulates retention time and peak metrics as csv/pdf outputs) — https://github.com/JohnsonDylan/MassQLab
- **MassQL** (Domain-specific query language used to extract MS1 extracted-ion retention times per analyte (e.g., MS1MZ=207.1418:TOLERANCEPPM=2.5 with RTMIN/RTMAX windows)) — https://github.com/mwang87/MassQueryLanguage
- **MSConvert** (Converts vendor .raw SST acquisition files to mzML with default settings before querying) — https://proteowizard.sourceforge.io/
- **Python** (Language used to compute per-analyte relative standard deviations from the MassQLab result tables (MassQLab developed using Python 3.9))

## Examples

```
py src\MassQLab_console.py  # after setting massqlab_config.json data_directory to SST mzML files and queryfile to named per-analyte MassQL queries; then compute RSD from ms1_RT_analysis_df.csv
```

## Evaluation signals

- Per-analyte RSD values must fall within the reported 0.5%–2.3% range (Table S2); minimum ≈ 0.5% and maximum ≈ 2.3%
- Each SST analyte (six deuterated acylcarnitines) yields a valid detection meeting MassQLab peak detection criteria in every expected acquisition file
- Detected retention times lie inside the RT windows defined in the MassQL queries (e.g., RTMIN=1.0 AND RTMAX=1.2 for m/z 207.1418)
- Result tables (ms1_raw_df.csv, ms1_RT_analysis_df.csv) exist for all query × file combinations with no missing entries
- Statistics are computed over the full assessment scope: ~6 months, two instrument platforms (Orbitrap Fusion and Fusion Lumos), and five operators

## Limitations

- Intensity and peak shape were not evaluated in the illustrative SST assessment
- RSD acceptability relies on uncontrolled variables (different personnel, columns, and LC solvents) being present across acquisitions; tighter or different thresholds may be needed for a single-operator setup
- Only 2% of surveyed untargeted metabolomics papers used system suitability samples, so cross-study comparability of SST metrics is limited
- MassQLab performance and available functions are still to be refined/expanded by the developers

## Evidence

- [results] RSD range 0.5–2.3%: "Statistical assessment of retention time, Table S2, yielded a relative standard deviation (RSD) in all SST analytes ranging from 0.5% to 2.3%"
- [results] Acquisition scope: [evidence span withheld — anchor status: not_found]
- [methods] Example MS1 query: "QUERY scaninfo(MS1DATA) FILTER MS1MZ=207.1418:TOLERANCEPPM=2.5 AND RTMIN=1.0 AND RTMAX=1.2"
- [intro] Query-to-table workflow: "MassQLab applies a series of queries (written in the language of MassQL) to a directory containing mass spectrometry data in mzML format. Results are tabulated and saved as images and csv files."
- [readme] RT stats output file: "ms1_RT_analysis_df.csv: RT stats vs. query RT"
- [results] Intensity/peak shape excluded: "Note, the intensity and peak shape were not evaluated."
