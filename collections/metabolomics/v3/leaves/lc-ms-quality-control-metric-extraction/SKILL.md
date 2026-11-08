---
name: lc-ms-quality-control-metric-extraction
description: Use when Thermo .raw files are being acquired across a specimen sequence
  and per-specimen IS-derived RT, intensity, and Δm/z values must be tabulated to
  expose acute QC failures or gradual drift over hours, days, months, or years.
when_to_use_negative:
- Data is in a vendor format other than Thermo RAW — MSConvert/mzML conversion is
  supported but the pipeline has only been tested extensively on Thermo acquisition
  sequences and RAW files, so other vendors may hit bugs
- The goal is deep post-hoc statistical analysis of a completed feature table rather
  than continuous, per-file QC monitoring during acquisition
- Running on a platform without MSConvert and MS-DIAL installed manually (e.g., a
  headless Linux cluster); Rapid QC-MS was designed for Windows because of these dependencies
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Rapid QC-MS
  role: Python pipeline (compatible with Python 3.8-3.11) that watches for new raw
    files, runs QC, and updates the interactive dashboard with IS RT, m/z, and intensity
    metrics
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts each vendor .raw file into open mzML format before routing it to
    the data processing pipeline (version 3.0.23332-7faa7f422)
- name: MS-DIAL
  role: Processes mzML files against the .msp library to extract internal standard
    peak metrics (ver.4.9.22121823; must be installed separately)
- name: MSConvert
  role: Dependency that enables universal vendor-format support by converting raw
    data to mzML
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:5e80d9894d0b2c9efe7a35016fc5531ea4f5971b462d9718b0e514191988becb
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
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lc-ms-quality-control-metric-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-specimen internal standard (IS) quality metrics — retention time, intensity, and Δm/z — from LC-MS/MS raw files using the Rapid QC-MS pipeline (MSConvert → MS-DIAL) so that instrument sensitivity, in-run retention time shifts, and mass accuracy drift can be monitored during an analysis.

## When to use

Use when Thermo .raw files are being acquired across a specimen sequence and per-specimen IS-derived RT, intensity, and Δm/z values must be tabulated to expose acute QC failures or gradual drift over hours, days, months, or years.

## When NOT to use

- Data is in a vendor format other than Thermo RAW — MSConvert/mzML conversion is supported but the pipeline has only been tested extensively on Thermo acquisition sequences and RAW files, so other vendors may hit bugs
- The goal is deep post-hoc statistical analysis of a completed feature table rather than continuous, per-file QC monitoring during acquisition
- Running on a platform without MSConvert and MS-DIAL installed manually (e.g., a headless Linux cluster); Rapid QC-MS was designed for Windows because of these dependencies

## Inputs

- Thermo .raw files written during an LC-MS/MS acquisition sequence
- Thermo acquisition sequence .csv
- Internal standard spectral library (.msp)
- Supplementary example files from Zenodo (10.5281/zenodo.10525183) and example data from Metabolomics Workbench (DOI 10.21228/M8Z119)

## Outputs

- Per-specimen table of internal standard retention time, intensity, and Δm/z values across all IS targets, in acquisition order
- Browser-viewable interactive dashboard (Plotly Dash) updated minutes after each new raw file is written
- Optional Slack or email QC-fail notifications for specimens with poor analytical integrity

## How to apply

Launch the pipeline with the 'rapidqcms' command, which watches the acquisition directory so that minutes after each new .raw file is written it is converted to open mzML via MSConvert (version 3.0.23332-7faa7f422) and routed to MS-DIAL ver.4.9.22121823 for peak picking against a supplied .msp spectral library. Parse the MS-DIAL IS QC results to tabulate per-specimen retention time, intensity, and Δm/z for every internal standard target, in acquisition order, mapping each specimen to its instrument sensitivity, in-run RT shift, and mass accuracy drift dimensions. Configure QC checks and, optionally, Slack/email alerting so specimens with potentially poor analytical integrity are flagged as each file completes. Validate by confirming the dashboard updates with results for every new file and that the tabulated RT, intensity, and Δm/z values are consistent with the interactive browser-viewable visualizations across specimens.

## Related tools

- **Rapid QC-MS** (Python pipeline (compatible with Python 3.8-3.11) that watches for new raw files, runs QC, and updates the interactive dashboard with IS RT, m/z, and intensity metrics) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts each vendor .raw file into open mzML format before routing it to the data processing pipeline (version 3.0.23332-7faa7f422))
- **MS-DIAL** (Processes mzML files against the .msp library to extract internal standard peak metrics (ver.4.9.22121823; must be installed separately))
- **MSConvert** (Dependency that enables universal vendor-format support by converting raw data to mzML)

## Examples

```
rapidqcms
```

## Evaluation signals

- Dashboard updates within minutes of each new .raw file being written, with results spanning instrument sensitivity, in-run retention time shift, and mass accuracy drift dimensions
- Tabulated metrics contain one row per specimen in the run, in acquisition order, with RT, intensity, and Δm/z columns for all IS targets
- Values in the tabulated CSV are consistent with the interactive dashboard visualizations for the same specimens
- Specimens failing user-defined QC checks trigger configured Slack or email alerts
- Intermediate mzML files exist for each completed raw file, confirming the MSConvert conversion step ran before MS-DIAL processing

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs and issues
- MSConvert and MS-DIAL must be installed manually; the package was designed for Windows platforms, though MacOS users can still monitor/view run data
- Was initially developed for integration with Thermo Orbitrap workflows

## Evidence
