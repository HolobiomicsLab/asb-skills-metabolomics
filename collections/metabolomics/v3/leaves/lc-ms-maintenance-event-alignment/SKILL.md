---
name: lc-ms-maintenance-event-alignment
description: Use when you have time-ordered DIA and paired DDA QC runs from a monitored
  instrument with maintenance-event timestamps, and need to determine whether DIA-based
  QC metrics (e.g. MS2 intensity) respond more sensitively than DDA metrics (e.g.
  MS1 area) to system status changes.
when_to_use_negative:
- Input is a single file or cross-sectional data without longitudinal ordering — the
  skill requires consecutive-injection ratio-of-change computation against maintenance
  timestamps
- No maintenance-event timestamps exist for the instrument, since alignment of metric
  changes to recorded events cannot be performed
- The goal is general DIA file quality scoring (qualified/unqualified LC/MS prediction)
  rather than maintenance-event sensitivity analysis — the standalone iDIA-QC ML prediction
  mode is the right fit instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: GUI suite that ingests raw files, extracts the 15 QC metrics, exports metric
    matrices, and generates the longitudinal HTML report with ratio-of-change curves
  repo: https://github.com/guomics-lab/iDIA-QC
- name: msConvert
  role: Converts vendor raw files (.raw, .d, .wiff) to mzXML for metric extraction
- name: DIA-NN
  role: Analyzes DIA files (v1.7.15) to derive peptide and protein quantification
    used for DIA-side QC metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:c9667834f7a5bc7ba540c6af630adca7561723a070d2a92b6d952f65c8ff91bf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  repo_url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - Orbitrap
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lc-ms-maintenance-event-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns longitudinal LC-MS QC metric fluctuations (ratio-of-change between consecutive injections) with recorded maintenance-event timestamps on a monitored instrument, enabling sensitivity comparison of DIA- vs DDA-based QC metrics for detecting system faults.

## When to use

Use when you have time-ordered DIA and paired DDA QC runs from a monitored instrument with maintenance-event timestamps, and need to determine whether DIA-based QC metrics (e.g. MS2 intensity) respond more sensitively than DDA metrics (e.g. MS1 area) to system status changes.

## When NOT to use

- Input is a single file or cross-sectional data without longitudinal ordering — the skill requires consecutive-injection ratio-of-change computation against maintenance timestamps
- No maintenance-event timestamps exist for the instrument, since alignment of metric changes to recorded events cannot be performed
- The goal is general DIA file quality scoring (qualified/unqualified LC/MS prediction) rather than maintenance-event sensitivity analysis — the standalone iDIA-QC ML prediction mode is the right fit instead

## Inputs

- Paired vendor raw LC-MS files (.raw, .d, or .wiff) from consecutive DDA and DIA injections (e.g. mouse liver digest QC runs)
- Instrument type and instrument ID assignments per file
- Recorded maintenance-event timestamps for the monitored instrument (e.g. QE HF-X)
- Mouse spectral library (110,126 precursors, 10,542 protein groups) for DIA-NN analysis

## Outputs

- Ratio-of-change curves for each QC metric (peptide number, protein number, MS1 area, MS2 intensity) aligned to maintenance-event timestamps
- iDIA-QC HTML report plotting metric ratio-of-change curves around maintenance events
- Exported QC metric matrices (per-file numerical metrics usable without R/Python)

## How to apply

Select paired vendor raw files (.raw, .d, or .wiff) in the iDIA-QC Input pane, assigning each file its instrument type and instrument ID, then convert to mzXML with msConvert. Analyze DIA files with DIA-NN (v1.7.15) against the mouse spectral library (110,126 precursors, 10,542 protein groups) using 1% FDR filtration at both peptide precursor and protein levels. Extract per-file numerical QC metrics — peptide number, protein number, MS1 area for DDA files, and MS2 intensity for DIA files — from the mzXML and DIA-NN outputs. Compute the ratio-of-change of each metric between consecutive injections and align those values with the recorded maintenance-event timestamps for the instrument. Finally, run iDIA-QC to export the metric matrices and generate the HTML report plotting ratio-of-change curves around the maintenance events.

## Related tools

- **iDIA-QC** (GUI suite that ingests raw files, extracts the 15 QC metrics, exports metric matrices, and generates the longitudinal HTML report with ratio-of-change curves) — https://github.com/guomics-lab/iDIA-QC
- **msConvert** (Converts vendor raw files (.raw, .d, .wiff) to mzXML for metric extraction)
- **DIA-NN** (Analyzes DIA files (v1.7.15) to derive peptide and protein quantification used for DIA-side QC metrics)

## Examples

```
iDIA-QC: select .raw/.d/.wiff files in the Input pane → assign instrument type/ID → Run to convert (msConvert), analyze DIA with DIA-NN v1.7.15, and export the HTML ratio-of-change report
```

## Evaluation signals

- Reproduced ratio-of-change curves around maintenance events show DIA-based LC-MS/MS consensus QC metrics exhibiting higher sensitivity than DDA-based QC metrics in detecting changes in LC-MS status
- DIA-NN results pass 1% FDR filtration at both peptide precursor and protein levels
- Per-file metric matrices are exported for each monitored file (peptide number, protein number, MS1 area for DDA, MS2 intensity for DIA)
- HTML report renders ratio-of-change curves correctly aligned to the recorded maintenance-event timestamps on the QE HF-X instrument
- Each raw file was assigned an instrument type and instrument ID before conversion and analysis

## Limitations

- Findings are grounded in mouse liver digest QC data from 21 mass spectrometers over 31 months; generalization of the workflow to other QC and experimental samples requires further investigation
- Study could not cover all LC and MS instruments, especially newly emerging ones
- This skill addresses fault-sensitivity comparison, not the AI qualified/unqualified prediction whose hyperparameters and train/test split composition are not described

## Evidence
