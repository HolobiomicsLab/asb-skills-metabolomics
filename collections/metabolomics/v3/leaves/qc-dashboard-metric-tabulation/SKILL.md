---
name: qc-dashboard-metric-tabulation
description: Use when raw LC-MS/MS specimen files from a monitored analysis need their
  internal standard RT, intensity, and Δm/z values tabulated in acquisition order
  to expose gradual drift.
when_to_use_negative:
- Data is already a processed feature table or QC report — this skill is for deriving
  IS QC metrics from raw acquisition files, not re-summarizing existing results
- Acquisition uses Agilent, Bruker, Sciex, or Waters sequences — Rapid QC-MS has only
  been tested extensively on Thermo acquisition sequences and Thermo RAW files, so
  IS tabulation for those vendors may fail
- The goal is acute failure alerting rather than gradual drift analysis — use Rapid
  QC-MS's messaging-app alerts instead of the tabulation workflow
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Rapid QC-MS
  role: Orchestrates the pipeline ('rapidqcms' command), parses IS QC results, and
    renders the tabulated RT, intensity, and Δm/z metrics on the interactive dashboard
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts each vendor .raw file into open mzML format before routing it to
    the data processing pipeline
- name: MS-DIAL
  role: Processes mzML files against the supplied .msp library to extract internal
    standard peak metrics per specimen
- name: Python
  role: Language of Rapid QC-MS (compatible with versions 3.8–3.11); used to parse
    IS QC results and write the per-specimen CSV table
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:c26581479a54624fdcfbf54bafda5413418db61abb8a47d4de4e17db2372994b
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# qc-dashboard-metric-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulate per-specimen internal standard retention time, intensity, and Δm/z values in acquisition order from Rapid QC-MS processing, so that gradual drifts in instrument sensitivity, in-run retention time shifts, and mass accuracy become visible in the interactive dashboard.

## When to use

Use when raw LC-MS/MS specimen files from a monitored analysis need their internal standard RT, intensity, and Δm/z values tabulated in acquisition order to expose gradual drift.

## When NOT to use

- Data is already a processed feature table or QC report — this skill is for deriving IS QC metrics from raw acquisition files, not re-summarizing existing results
- Acquisition uses Agilent, Bruker, Sciex, or Waters sequences — Rapid QC-MS has only been tested extensively on Thermo acquisition sequences and Thermo RAW files, so IS tabulation for those vendors may fail
- The goal is acute failure alerting rather than gradual drift analysis — use Rapid QC-MS's messaging-app alerts instead of the tabulation workflow

## Inputs

- Vendor raw LC-MS/MS data files (.raw, e.g. Thermo RAW files)
- Sequence table (.csv) specifying acquisition order of specimens
- Internal standard spectral library (.msp)
- MSConvert-converted mzML files

## Outputs

- CSV table with one row per specimen in acquisition order, mapping each specimen to IS-derived retention time, intensity, and Δm/z values across all IS targets
- Browser-viewable dashboard plots of IS RT, m/z, and intensity across samples for drift monitoring

## How to apply

Download the supplementary example files from Zenodo (10.5281/zenodo.10525183: sequence .csv, library .msp, and .raw files) and start the pipeline with the 'rapidqcms' command. MSConvert (3.0.23332–7faa7f422) converts each vendor .raw file into open mzML format before routing it to the processing pipeline, which requires MS-DIAL ver.4.9.22121823 to process the mzML files against the supplied .msp library. Parse the resulting IS QC results to extract retention time, intensity, and Δm/z for every internal standard target, tabulating one row per specimen in acquisition order. Emit the tabulation as a CSV mapping each specimen to its IS-derived RT, intensity, and mass-accuracy values so the dashboard can render gradual drifts across hours to months of analysis. Rationale: within minutes of a new raw file being written, these three metrics correspond directly to the dashboard's instrument sensitivity, in-run retention time shift, and mass accuracy drift performance dimensions.

## Related tools

- **Rapid QC-MS** (Orchestrates the pipeline ('rapidqcms' command), parses IS QC results, and renders the tabulated RT, intensity, and Δm/z metrics on the interactive dashboard) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts each vendor .raw file into open mzML format before routing it to the data processing pipeline)
- **MS-DIAL** (Processes mzML files against the supplied .msp library to extract internal standard peak metrics per specimen)
- **Python** (Language of Rapid QC-MS (compatible with versions 3.8–3.11); used to parse IS QC results and write the per-specimen CSV table) — https://github.com/czbiohub-sf/Rapid-QC-MS/

## Examples

```
rapidqcms
```

## Evaluation signals

- The CSV exists and contains one row per specimen in the example run, ordered by acquisition sequence
- The table includes RT, intensity, and Δm/z columns whose values are consistent with the dashboard's instrument sensitivity, in-run retention time shift, and mass accuracy drift dimensions
- Internal standard peak metrics match what MS-DIAL ver.4.9.22121823 reports when processing the mzML files against the supplied .msp library
- Dashboard plots update within minutes of a new data file being written, reflecting the newly tabulated metrics

## Limitations

- Rapid QC-MS was designed for Windows platforms due to MSConvert and MS-DIAL dependencies; MacOS users can only monitor/view instrument run data
- Only tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs and issues
- MSConvert and MS-DIAL must be downloaded and installed manually; MS-DIAL ver.4.9.22121823 is required

## Evidence
