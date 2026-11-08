---
name: mzml-peak-feature-annotation
description: Use when vendor .raw files need to be converted to mzML and matched against
  a .msp library to tabulate internal standard RT, intensity, and mass-accuracy metrics
  per specimen during QC.
when_to_use_negative:
- Input is already a processed feature table or mzML — annotation via MSConvert raw
  conversion is unnecessary
- The goal is de novo feature detection without a .msp spectral library of internal
  standard targets
- Non-Thermo vendor sequences (.raw from Agilent, Bruker, Sciex, Waters) where support
  is explicitly untested and bugs are expected
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts vendor raw acquired data into open mzML format before routing to
    the data processing pipeline
- name: MS-DIAL ver.4.9.221218
  role: Processes mzML files against the .msp library to extract internal standard
    peak metrics; must be downloaded and installed separately
- name: Rapid QC-MS
  role: Python pipeline (versions 3.8–3.11) launched via the 'rapidqcms' command that
    orchestrates conversion, MS-DIAL processing, and IS QC result parsing
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:ac6e3fb615d902436c41d816b64792b9d6296fc9907c22b009de099452bbf0b1
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
  - MS-DIAL ver.4.9.221218
  - MSConvert
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# mzml-peak-feature-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts vendor raw LC-MS/MS files into open mzML format with MSConvert and annotates peaks against a spectral library (.msp) with MS-DIAL, yielding per-specimen internal standard retention time, intensity, and Δm/z metrics for drift monitoring.

## When to use

Use when vendor .raw files need to be converted to mzML and matched against a .msp library to tabulate internal standard RT, intensity, and mass-accuracy metrics per specimen during QC.

## When NOT to use

- Input is already a processed feature table or mzML — annotation via MSConvert raw conversion is unnecessary
- The goal is de novo feature detection without a .msp spectral library of internal standard targets
- Non-Thermo vendor sequences (.raw from Agilent, Bruker, Sciex, Waters) where support is explicitly untested and bugs are expected

## Inputs

- Vendor LC-MS/MS raw files (e.g., Thermo .raw)
- Sequence .csv file defining the acquisition order
- Spectral library in .msp format containing internal standard targets

## Outputs

- Converted mzML files (one per raw file)
- Per-specimen internal standard peak metrics: retention time, intensity, and Δm/z
- CSV table mapping each specimen to IS-derived RT, intensity, and mass-accuracy values in acquisition order

## How to apply

Launch the Rapid QC-MS pipeline with the 'rapidqcms' command; MSConvert (3.0.23332–7faa7f422) converts each vendor raw file into open mzML format before routing it to the data processing pipeline. Process the mzML files through MS-DIAL (ver.4.9.221218) against the supplied .msp library to extract internal standard peak metrics for each specimen. Parse the IS QC results to tabulate per-specimen retention time, intensity, and Δm/z values across all IS targets in acquisition order. Because MSConvert normalizes vendor formats to mzML, the approach should in principle work for any vendor, but only Thermo RAW files and Thermo acquisition sequences have been extensively tested. The tabulated metrics map each specimen to IS-derived RT, intensity, and mass-accuracy values so gradual drifts over hours, days, months, or years become visible in the dashboard.

## Related tools

- **MSConvert 3.0.23332-7faa7f422** (Converts vendor raw acquired data into open mzML format before routing to the data processing pipeline)
- **MS-DIAL ver.4.9.221218** (Processes mzML files against the .msp library to extract internal standard peak metrics; must be downloaded and installed separately)
- **Rapid QC-MS** (Python pipeline (versions 3.8–3.11) launched via the 'rapidqcms' command that orchestrates conversion, MS-DIAL processing, and IS QC result parsing) — https://github.com/czbiohub-sf/Rapid-QC-MS

## Examples

```
rapidqcms
```

## Evaluation signals

- Each specimen in the run yields one row in the tabulated CSV, ordered by acquisition sequence
- The table includes RT, intensity, and Δm/z columns whose values are consistent with the dashboard's instrument sensitivity, in-run retention time shift, and mass accuracy drift dimensions
- Every raw file is successfully converted to mzML by MSConvert before MS-DIAL processing
- Internal standard identifications derive from the supplied .msp library via MS-DIAL ver.4.9.221218

## Limitations

- Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert and MS-DIAL; MacOS users can only monitor/view instrument run data
- Only tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs
- MSConvert and MS-DIAL must be installed manually since Python dependencies alone are installed automatically during setup

## Evidence
