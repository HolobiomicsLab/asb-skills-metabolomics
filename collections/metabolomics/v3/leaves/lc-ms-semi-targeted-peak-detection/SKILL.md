---
name: lc-ms-semi-targeted-peak-detection
description: Use when the goal is to reproduce automated semi-targeted quantification
  of predefined analytes and internal standards from vendor DDA LC-MS files (Thermo
  .raw, SCIEX .wiff) without a spectral library or manual curation.
when_to_use_negative:
- The analysis requires curated MS/MS spectra or identification of unknown metabolites
  — this skill targets semi-targeted quantification with a spectrum-free compound
  library only.
- Input data were acquired in DIA/SWATH, MRM, or PRM mode — MetaPro supports MS data
  acquired with the DDA method for this workflow.
- A finished feature/peak-area table already exists and only downstream normalization
  or statistics is needed.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetaPro v1.1.8
  role: Web application that imports Aird files, assigns analyte/IS libraries and
    Methods per batch, and runs Target Search and Export Report for semi-targeted
    quantification.
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro 1.1
  role: GUI client converting vendor files (Thermo .raw, SCIEX .wiff) to the computation-oriented
    Aird format (.aird plus . index) in DDA mode before MetaPro import.
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Started by the MetaPro install scripts as part of the runtime stack (and used
    by AirdPro for distributed batch conversion tasks).
- name: MongoDB
  role: Started by the MetaPro install scripts as part of the runtime stack.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:2d1d3d3825731aa6dfe626b8a3756ee31e7d0f31f1c35c62fa10d167aac0973c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdPro 1.1
  - MetaPro
  - MetaPro v1.1.8
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# lc-ms-semi-targeted-peak-detection

## Summary

Runs MetaPro's fully automatic semi-targeted quantification on LC-MS DDA data converted to Aird format, using a compound library with no spectra and no manual inspection, to produce reproducible peak areas on benchmark datasets (TripleTOF 6600, Q Exactive HF).

## When to use

Use when the goal is to reproduce automated semi-targeted quantification of predefined analytes and internal standards from vendor DDA LC-MS files (Thermo .raw, SCIEX .wiff) without a spectral library or manual curation.

## When NOT to use

- The analysis requires curated MS/MS spectra or identification of unknown metabolites — this skill targets semi-targeted quantification with a spectrum-free compound library only.
- Input data were acquired in DIA/SWATH, MRM, or PRM mode — MetaPro supports MS data acquired with the DDA method for this workflow.
- A finished feature/peak-area table already exists and only downstream normalization or statistics is needed.

## Inputs

- Vendor LC-MS DDA raw files (Thermo .raw, SCIEX .wiff) from TripleTOF 6600 and Q Exactive HF platforms
- Analyte library table (.txt/.tsv with header containing at least mz and rt)
- Internal standard library table (.txt/.tsv with header containing at least mz and rt)
- Converted Aird files (.aird and . index) arranged in project/batch folder hierarchy

## Outputs

- Quantification report (.xlsx) with peak areas per compound per sample from Export Report
- Per-compound log-transformed fold changes of SB relative to SA peak areas

## How to apply

Convert vendor files (Thermo .raw, SCIEX .wiff) to Aird format (.aird plus . index) with AirdPro 1.1 in DDA mode, since MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method. Arrange converted files under a project/batch folder hierarchy and import them via the Scan Files function of MetaPro v1.1.8, which scans folder structure for multi-batch import. Upload analyte and internal standard libraries (.txt/.tsv whose first line contains at least the mz and rt items) on the Library page with Library Type set to ANALYTES and IS, then create a Method per instrument platform (AB SCIEX TripleTOF 6600 and Thermo Q Exactive HF) using the preset recommended parameters and assign libraries and method to each batch. Launch Target Search on all batches and, to reproduce the paper's first run, skip any manual QC inspection or library curation (no Reset integration params, Reset TargetRT, or Manual Integration). Export the overview via Export Report to obtain peak areas as an .xlsx file, then compute per-compound log-transformed fold changes of signal in blank (SB) relative to analyte sample (SA) to match the Fig. 4B/4F comparison conditions.

## Related tools

- **MetaPro v1.1.8** (Web application that imports Aird files, assigns analyte/IS libraries and Methods per batch, and runs Target Search and Export Report for semi-targeted quantification.) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro 1.1** (GUI client converting vendor files (Thermo .raw, SCIEX .wiff) to the computation-oriented Aird format (.aird plus . index) in DDA mode before MetaPro import.) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Started by the MetaPro install scripts as part of the runtime stack (and used by AirdPro for distributed batch conversion tasks).)
- **MongoDB** (Started by the MetaPro install scripts as part of the runtime stack.)

## Examples

```
bash MetaproStart.sh  # then browse localhost:8080, Scan Files -> Load library (ANALYTES/IS) -> Create Method -> Target Search -> Export Report
```

## Evaluation signals

- Result distribution of the automatic first run (no spectra, no manual inspection) matches the post-manual-inspection distribution on the benchmark datasets, indicating quantification accuracy and stability.
- Aird output contains both .aird data file and . index file with the same file name in the same directory, so AirdScanUtil can scan both.
- MetaPro scans the project/batch folder hierarchy and imports all converted files via Scan Files without per-file manual loading.
- Exported .xlsx report contains peak areas for every compound in the analyte and IS libraries across all batches.
- Computed log-transformed SB/SA fold changes reproduce the Fig. 4B/4F comparison conditions for both the TripleTOF and QE HF datasets.

## Limitations

- MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method; other acquisition modes fall outside this workflow.
- The automatic run skips manual QC inspection; without curated spectra, identification accuracy depends on library quality, and manual inspection (Reset integration params, Reset TargetRT, Manual Integration) is available but deliberately not used in this reproduction.
- MetaPro is for academic use only; commercial use requires contacting the authors.
- The Aird index and data files must share the same directory and file name (differing only in suffix) or they cannot be scanned.

## Evidence

- [methods] MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method.: "MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
- [methods] Convert files and arrange directory: "**Step 2: File directory arrangement** MetaPro enables efficient multi-batch files import by scanning folder structure"
- [methods] Load library definition: "#### 4.2.2 Load library Enter Library page, click `Create Library` at the top."
- [methods] Target Search trigger: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
- [readme] Aird file naming rule: [evidence span withheld — anchor status: external_doc]
- [readme] Versioned downloads: "[AirdPro v1.1](https://github.com/CSi-Studio/AirdPro/releases/tag/1.1)"
