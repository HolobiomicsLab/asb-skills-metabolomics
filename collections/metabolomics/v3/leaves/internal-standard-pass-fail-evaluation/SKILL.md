---
name: internal-standard-pass-fail-evaluation
description: Use when acquiring large untargeted LC-MS/MS runs (e.g., Thermo RAW files
  with a sequence .csv and an .msp internal-standard library) and you need automatic,
  per-specimen Pass/Fail/Warn status for every internal standard target as soon as
  mzML-converted data arrive.
when_to_use_negative:
- When the goal is a one-shot batch analysis of already-completed, manually curated
  data rather than continuous QC during an instrument run — Rapid QC-MS is oriented
  toward monitoring specimens as raw files are written.
- When working exclusively with non-Thermo vendor formats (Agilent, Bruker, Sciex,
  Waters) — the README states Rapid QC-MS 'has only been tested extensively on Thermo
  Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files' and
  expects bugs with other vendor formats.
- On non-Windows platforms where MSConvert and MS-DIAL cannot be installed — those
  dependencies are required for the raw-file-to-QC-assignment pipeline.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: Rapid QC-MS
  role: Primary software that runs the per-specimen, per-IS Pass/Fail/Warn evaluation
    and renders the results on an interactive dashboard
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts vendor RAW files into open mzML format before routing them to the
    data-processing pipeline (version 3.0.23332-7faa7f422 used in the paper)
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL
  role: Performs feature extraction against the .msp internal-standard library to
    supply the RT/m/z/intensity values that the Pass/Fail/Warn logic evaluates (version
    ver.4.9.22121823 used in the paper)
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Python
  role: Runtime environment (3.8–3.11) in which Rapid QC-MS is installed and executed
    via 'rapidqcms'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:a32ee00cbd291d991f00e8d4293337c405164314cd0b8003ce6a0373132ef63f
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
  - MSConvert 3.0.23332-7faa7f422
  - Python
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# internal-standard-pass-fail-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Rapid QC-MS capability that assigns each internal standard (IS) target within a specimen a Pass, Fail, or Warn designation — with missing IS marked Fail — so per-specimen LC-MS/MS analytical integrity can be judged within minutes of a raw file being written.

## When to use

Use when acquiring large untargeted LC-MS/MS runs (e.g., Thermo RAW files with a sequence .csv and an .msp internal-standard library) and you need automatic, per-specimen Pass/Fail/Warn status for every internal standard target as soon as mzML-converted data arrive.

## When NOT to use

- When the goal is a one-shot batch analysis of already-completed, manually curated data rather than continuous QC during an instrument run — Rapid QC-MS is oriented toward monitoring specimens as raw files are written.
- When working exclusively with non-Thermo vendor formats (Agilent, Bruker, Sciex, Waters) — the README states Rapid QC-MS 'has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files' and expects bugs with other vendor formats.
- On non-Windows platforms where MSConvert and MS-DIAL cannot be installed — those dependencies are required for the raw-file-to-QC-assignment pipeline.

## Inputs

- Vendor raw data files (e.g., Thermo .raw files)
- Thermo acquisition sequence .csv
- Internal standard library .msp file
- mzML-converted data (produced internally by MSConvert)

## Outputs

- Per-specimen, per-internal-standard Pass / Fail / Warn QC designation (missing IS marked Fail)
- Browser-viewable dashboard with interactive visualization of IS retention time, m/z, and intensity across samples
- Slack or email QC-fail notifications

## How to apply

Launch Rapid QC-MS (enter 'rapidqcms' in the terminal) and configure a new QC job pointing at the vendor raw files, the Thermo acquisition sequence .csv, and the .msp library of internal standards. MSConvert (3.0.23332-7faa7f422) converts each Thermo RAW file into open mzML format before routing it to the data-processing pipeline, and MS-DIAL (ver.4.9.22121823) performs feature extraction against the .msp library. For each specimen, Rapid QC-MS then evaluates the extracted IS retention time, m/z, and intensity against user-defined quality-control checks and assigns every internal standard target a Pass, Fail, or Warn designation; IS targets absent from the extracted features are marked Fail. The per-specimen IS assignments are rendered on a browser-viewable dashboard minutes after the new data file is written and can trigger Slack or email notifications, allowing users to identify specimens with potentially poor analytical integrity without manually curating raw data. Validation consists of comparing the regenerated per-specimen IS Pass/Fail/Warn assignments against the determinations reported in the paper's Specimen Table for the Zenodo 10.5281/zenodo.10525183 example files.

## Related tools

- **Rapid QC-MS** (Primary software that runs the per-specimen, per-IS Pass/Fail/Warn evaluation and renders the results on an interactive dashboard) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts vendor RAW files into open mzML format before routing them to the data-processing pipeline (version 3.0.23332-7faa7f422 used in the paper)) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL** (Performs feature extraction against the .msp internal-standard library to supply the RT/m/z/intensity values that the Pass/Fail/Warn logic evaluates (version ver.4.9.22121823 used in the paper)) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Python** (Runtime environment (3.8–3.11) in which Rapid QC-MS is installed and executed via 'rapidqcms') — https://github.com/czbiohub-sf/Rapid-QC-MS

## Examples

```
py -m pip install rapid-qc-ms && rapidqcms
```

## Evaluation signals

- Every internal standard target in every specimen receives one of exactly three designations: Pass, Fail, or Warn — no target is left unassigned.
- Any internal standard absent from the extracted features for a specimen is assigned Fail (the documented default for missing IS).
- Per-specimen IS assignments reproduced from the Zenodo 10.5281/zenodo.10525183 example files match the determinations reported in the paper's Specimen Table.
- The dashboard updates within minutes after a new data file is written, confirming synchronous (in-run) evaluation rather than post-hoc batch QC.
- Slack or email notifications fire when a specimen's IS evaluation yields a Fail (or Warn) condition that violates the configured quality-control checks.

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; processing data of other vendor formats may contain bugs.
- MSConvert and MS-DIAL must be installed manually — they are not installed automatically by 'py -m pip install rapid-qc-ms'.
- Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert and MS-DIAL; MacOS users can only monitor/view run data.
- Support for Agilent, Bruker, Sciex, or Waters acquisition sequences and data files is not yet developed (collaboration invited).

## Evidence

- [results] Rapid QC-MS assigns each internal standard target within a specimen a Pass, Fail, or Warn designation (missing IS are marked Fail): [evidence span withheld — anchor status: not_found]
- [methods] MS-DIAL performs feature extraction against the .msp library: [evidence span withheld — anchor status: not_found]
