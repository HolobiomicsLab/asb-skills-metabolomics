---
name: internal-standard-drift-trend-analysis
description: Use when a raw-file series (e.g. Thermo .raw from a Thermo acquisition
  sequence) has been processed through MS-DIAL against an .msp IS library and you
  need to detect gradual drift in internal standard RT, intensity, or Δm/z across
  specimens.
when_to_use_negative:
- Data are non-Thermo vendor acquisition sequences (Agilent, Bruker, Sciex, Waters),
  which are not yet tested and may contain bugs
- Input is already a processed feature table or QC report rather than raw vendor/mzML
  data requiring MS-DIAL identification against the IS library
- The goal is batch post-hoc statistical drift modeling rather than synchronous, file-by-file
  QC monitoring during an instrument run
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: Rapid QC-MS
  role: Orchestrates the pipeline and renders the interactive dashboard of IS RT,
    m/z, and intensity used to expose drift
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts each newly written raw file into open mzML format before routing
    to the data processing pipeline
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL
  role: Processes mzML files against the .msp IS library to extract per-specimen IS
    peak metrics (RT, intensity, Δm/z)
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
- name: Python
  role: Runtime for Rapid QC-MS (versions 3.8–3.11) and for parsing/tabulating IS
    QC results into the drift CSV
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:2d10f01a4175c16eb41ee6891859ab8372930f107a1782aa281fb6e2cdfedb4b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  repo_url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL
  - MSConvert
  - Python
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# internal-standard-drift-trend-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulates per-specimen internal standard (IS) retention time, intensity, and Δm/z values from MS-DIAL-processed LC-MS/MS runs and inspects their trajectories over acquisition order via the Rapid QC-MS interactive dashboard, exposing gradual instrument sensitivity, retention-time-shift, and mass-accuracy drift.

## When to use

Use when a raw-file series (e.g. Thermo .raw from a Thermo acquisition sequence) has been processed through MS-DIAL against an .msp IS library and you need to detect gradual drift in internal standard RT, intensity, or Δm/z across specimens.

## When NOT to use

- Data are non-Thermo vendor acquisition sequences (Agilent, Bruker, Sciex, Waters), which are not yet tested and may contain bugs
- Input is already a processed feature table or QC report rather than raw vendor/mzML data requiring MS-DIAL identification against the IS library
- The goal is batch post-hoc statistical drift modeling rather than synchronous, file-by-file QC monitoring during an instrument run

## Inputs

- Thermo .raw files from an LC-MS/MS acquisition sequence
- Sequence .csv file defining the run order
- Internal standard library in .msp format
- mzML files converted from vendor raw data via MSConvert

## Outputs

- CSV table with one row per specimen mapping it to IS-derived retention time, intensity, and Δm/z values
- Interactive browser-based dashboard plots of IS RT, m/z, and intensity across samples in acquisition order

## How to apply

Download the Zenodo supplementary example files (10.5281/zenodo.10525183: sequence .csv, library .msp, additional .raw files) plus the Metabolomics Workbench example data (DOI 10.21228/M8Z119). Launch Rapid QC-MS with the 'rapidqcms' command; it converts each completed raw file into open mzML via MSConvert (3.0.23332–7faa7f422) and routes it to MS-DIAL ver.4.9.22121823, which matches specimens against the supplied .msp library to extract IS peak metrics. Parse the IS QC results to tabulate per-specimen retention time, intensity, and Δm/z for every IS target in acquisition order, writing a CSV mapping each specimen to its RT, intensity, and mass-accuracy values. Then inspect the interactive Plotly Dash browser dashboard, which is updated minutes after each new raw file is written, to visualize the three drift dimensions (instrument sensitivity, in-run retention time shift, mass accuracy drift) and distinguish acute deviations from gradual drift over hours, days, months, or years.

## Related tools

- **Rapid QC-MS** (Orchestrates the pipeline and renders the interactive dashboard of IS RT, m/z, and intensity used to expose drift) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts each newly written raw file into open mzML format before routing to the data processing pipeline) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL** (Processes mzML files against the .msp IS library to extract per-specimen IS peak metrics (RT, intensity, Δm/z)) — http://prime.psc.riken.jp/compms/msdial/main.html
- **Python** (Runtime for Rapid QC-MS (versions 3.8–3.11) and for parsing/tabulating IS QC results into the drift CSV) — https://github.com/czbiohub-sf/Rapid-QC-MS

## Examples

```
rapidqcms
```

## Evaluation signals

- The CSV table exists and contains exactly one row per specimen in the example run
- The table includes RT, intensity, and Δm/z columns whose values are consistent with the dashboard's instrument sensitivity, in-run retention time shift, and mass accuracy drift dimensions
- Specimens appear in acquisition order so that gradual drift over the course of the analysis is visible rather than scrambled
- Dashboard plots update within minutes of each new raw file being written, matching the synchronous-QC behavior described in the article

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs
- Designed for Windows platforms because of MSConvert and MS-DIAL dependencies; MacOS users can only monitor/view run data
- Dependencies such as MSConvert and MS-DIAL must be installed manually and are not pip-installed with the package

## Evidence

- [readme] Interactive data visualization of internal standard retention time, m/z, and intensity across samples: "**Interactive data visualization** of internal standard retention time, _m/z_, and intensity across samples"
- [abstract] Dashboard updates minutes after data files are written: "Minutes after a new data file is written, a browser-viewable dashboard is updated with quality control results"
- [methods] MSConvert converts raw data to mzML before pipeline: "MSConvert converts raw acquired data into open mzML format before routing it to the data processing pipeline"
- [abstract] Interactive visualizations reveal acute deviations and gradual drift: "Rapid QC-MS provides interactive visualizations that help users recognize acute deviations in these performance metrics, as well as gradual drifts"
- [readme] Only tested extensively on Thermo instruments and raw files: "Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files"
