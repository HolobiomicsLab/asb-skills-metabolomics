---
name: lc-ms-retention-time-tracking
description: Use when a longitudinal series of DIA raw files (.wiff/.raw/.d) from
  a fixed instrument must be quantified consistently and per-precursor retention-time/intensity
  stability (CV) needs to be summarized over time.
when_to_use_negative:
- Input is already a cleaned, imputed feature table — go directly to CV computation
  instead of re-running conversion and DIA-NN
- You need QC metrics for DDA or SRM data — the article states existing DDA/SRM QC
  pipelines 'cannot be directly applied to large-scale DIA experiments' and iDIA-QC
  is DIA-specific
- One-off single-shot DIA runs without a longitudinal component — RT/CV stability
  tracking is designed for repeated injections over time (e.g., 31-day series)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: GUI pipeline that orchestrates conversion, quantification, QC-metric extraction,
    and longitudinal html-report output for DIA files from timsTOF, TripleTOF, and
    Orbitrap instruments
  repo: github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: Derives peptide-level and protein-level quantification from converted mzXML
    files under 1% FDR at precursor and protein levels (v1.7.15 in the paper)
- name: msConvert
  role: Converts vendor .wiff/.raw/.d raw files to mzXML (extracted precursor ion
    chromatograms) prior to DIA-NN analysis
- name: TripleTOF
  role: SCIEX instrument family whose .wiff DIA acquisitions (e.g., 31-day WMLD series)
    form the longitudinal input for precursor CV tracking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:8f98428b64c1e3776ad48c4f7676909725ae73b9e5a69cc88549f5fba25faadb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - TripleTOF
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# lc-ms-retention-time-tracking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tracks longitudinal retention-time and intensity stability of selected DIA peptide precursors (e.g., 33 model-selected precursors on a TripleTOF 5600+) by processing 31-day .wiff DIA series through msConvert and DIA-NN and computing per-precursor CVs, as practiced in the iDIA-QC workflow.

## When to use

Use when a longitudinal series of DIA raw files (.wiff/.raw/.d) from a fixed instrument must be quantified consistently and per-precursor retention-time/intensity stability (CV) needs to be summarized over time.

## When NOT to use

- Input is already a cleaned, imputed feature table — go directly to CV computation instead of re-running conversion and DIA-NN
- You need QC metrics for DDA or SRM data — the article states existing DDA/SRM QC pipelines 'cannot be directly applied to large-scale DIA experiments' and iDIA-QC is DIA-specific
- One-off single-shot DIA runs without a longitudinal component — RT/CV stability tracking is designed for repeated injections over time (e.g., 31-day series)

## Inputs

- Longitudinal DIA raw files (.wiff from TripleTOF 5600+, or .raw/.d from other vendors)
- List of pre-selected peptide precursors with expected retention times
- Spectral library used by DIA-NN for precursor identification and quantification

## Outputs

- Run-by-precursor intensity/RT matrix
- Per-precursor CV statistics (median, mean, max, count above threshold) written to cv_statistics.csv
- iDIA-QC html report with longitudinal QC visualizations

## How to apply

Convert vendor .wiff raw files from the longitudinal DIA series to mzXML with msConvert, then analyze with DIA-NN (v1.7.15) applying 1% FDR at both peptide precursor and protein levels to derive peptide quantification. Clean the quantification matrix by removing features with >70% missing values and imputing remaining missing values with the per-feature minimum. Extract the retention times and intensities of the pre-selected peptide precursors (e.g., 33 precursors with no overlap between LC and MS panels) into a run-by-precursor matrix, then compute per-precursor CV (SD/mean × 100) across injections using pandas or R. Summarize the CV distribution (median, mean, count above threshold, maximum) and write it to cv_statistics.csv alongside the iDIA-QC html report and matrices, which provide a longitudinal visual summary of instrument quality.

## Related tools

- **iDIA-QC** (GUI pipeline that orchestrates conversion, quantification, QC-metric extraction, and longitudinal html-report output for DIA files from timsTOF, TripleTOF, and Orbitrap instruments) — github.com/guomics-lab/iDIA-QC
- **DIA-NN** (Derives peptide-level and protein-level quantification from converted mzXML files under 1% FDR at precursor and protein levels (v1.7.15 in the paper))
- **msConvert** (Converts vendor .wiff/.raw/.d raw files to mzXML (extracted precursor ion chromatograms) prior to DIA-NN analysis)
- **TripleTOF** (SCIEX instrument family whose .wiff DIA acquisitions (e.g., 31-day WMLD series) form the longitudinal input for precursor CV tracking)

## Examples

```
import pandas as pd; mat = pd.read_csv('precursor_matrix.csv', index_col=0); cv = (mat.std() / mat.mean() * 100); cv.to_csv('cv_statistics.csv')
```

## Evaluation signals

- Reproduced median CV of ~2.9% and mean CV of ~3.5% across the 33 tracked precursors over the 31-day series
- Exactly one precursor above the 5.8% CV threshold, with a maximum CV of ~11.2%
- Quantification matrix contains no features with >70% missing values and remaining NAs are imputed at the per-feature minimum
- DIA-NN run enforces 1% FDR at both peptide precursor and protein levels
- cv_statistics.csv plus iDIA-QC html report and matrices are all produced as outputs

## Limitations

- One precursor can exceed the 5.8% CV threshold (up to ~11.2%), so CV thresholds are not absolute pass/fail guarantees
- The QC approach was validated on specific instrument families (Orbitrap, TripleTOF, timsTOF Pro) and 'it is not possible to include all LC and MS instruments, particularly those emerging new instruments'
- Generalizing the workflow to other QC or experimental samples 'requires further investigation' per the article

## Evidence
