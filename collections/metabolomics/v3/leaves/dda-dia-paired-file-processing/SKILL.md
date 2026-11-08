---
name: dda-dia-paired-file-processing
description: Use when you hold paired vendor raw files (.raw, .d, .wiff) acquired
  in both DDA and DIA modes on the same instrument (e.g. QE HF-X) and need to compare
  DDA- vs DIA-based QC metric sensitivity around recorded maintenance events.
when_to_use_negative:
- Input is already an extracted metric/feature table — proceed directly to the ratio-of-change
  and maintenance-event alignment step instead of reprocessing raw files
- You only have DIA data without paired DDA acquisitions, so the DDA-vs-DIA sensitivity
  comparison cannot be made
- QC assessment for SRM/targeted experiments — the article notes SRM-based QC tools
  cannot be directly applied to large-scale quantitative proteomics experiments, and
  this skill targets DDA/DIA raw-file processing
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: iDIA-QC
  role: GUI used to select paired raw files, assign instrument type/ID, export metric
    matrices, and generate the HTML report of ratio-of-change curves
  repo: github.com/guomics-lab/iDIA-QC
- name: msConvert
  role: Converts vendor raw files (.raw, .d, .wiff) to mzXML for metric extraction
- name: DIA-NN
  role: Analyzes converted DIA files (v1.7.15) against the mouse spectral library
    to derive peptide and protein quantification with 1% FDR at precursor and protein
    levels
- name: Orbitrap
  role: Instrument series (e.g. QE HF-X) whose paired DDA/DIA acquisitions are processed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:8f736264956316aa4db66672114341243614ddbda230e924c28c698ad479a20b
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
  - Orbitrap
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# dda-dia-paired-file-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Paired DDA and DIA vendor raw files are converted to mzXML with msConvert, DIA files are quantified with DIA-NN (v1.7.15) at 1% FDR at both peptide precursor and protein levels, and numerical QC metrics are extracted per file so that DDA vs DIA sensitivity to LC-MS system faults can be compared around maintenance events.

## When to use

Use when you hold paired vendor raw files (.raw, .d, .wiff) acquired in both DDA and DIA modes on the same instrument (e.g. QE HF-X) and need to compare DDA- vs DIA-based QC metric sensitivity around recorded maintenance events.

## When NOT to use

- Input is already an extracted metric/feature table — proceed directly to the ratio-of-change and maintenance-event alignment step instead of reprocessing raw files
- You only have DIA data without paired DDA acquisitions, so the DDA-vs-DIA sensitivity comparison cannot be made
- QC assessment for SRM/targeted experiments — the article notes SRM-based QC tools cannot be directly applied to large-scale quantitative proteomics experiments, and this skill targets DDA/DIA raw-file processing

## Inputs

- Paired vendor raw mass spectrometry files in .raw, .d, or .wiff format (DDA and DIA acquisitions from the same instrument, e.g. QE HF-X)
- Recorded maintenance-event timestamps for the monitored instrument
- Mouse spectral library (110,126 precursors, 10,542 protein groups) for DIA-NN

## Outputs

- mzXML files converted from vendor raw formats via msConvert
- Per-file numerical QC metrics: peptide number, protein number, MS1 area (DDA) and MS2 intensity (DIA)
- Ratio-of-change curves of each metric between consecutive injections aligned to maintenance-event timestamps
- iDIA-QC metric matrices and HTML report plotting the curves around maintenance events

## How to apply

Select the paired vendor raw files in the iDIA-QC Input pane and assign each file its instrument type and instrument ID, then convert them to mzXML with msConvert. Analyze the DIA files with DIA-NN (v1.7.15) against a mouse spectral library (110,126 precursors, 10,542 protein groups), applying 1% FDR filtration at both peptide precursor and protein levels. Extract the numerical QC metrics per file — peptide number, protein number, and MS1 area for DDA files, MS2 intensity for DIA files — from the mzXML and DIA-NN outputs. Compute the ratio-of-change of each metric between consecutive injections and align those values with the recorded maintenance-event timestamps for the monitored instrument. Run iDIA-QC to export the metric matrices and generate the HTML report plotting ratio-of-change curves around the maintenance events, so DIA vs DDA responses to the same faults can be judged side by side.

## Related tools

- **iDIA-QC** (GUI used to select paired raw files, assign instrument type/ID, export metric matrices, and generate the HTML report of ratio-of-change curves) — github.com/guomics-lab/iDIA-QC
- **msConvert** (Converts vendor raw files (.raw, .d, .wiff) to mzXML for metric extraction)
- **DIA-NN** (Analyzes converted DIA files (v1.7.15) against the mouse spectral library to derive peptide and protein quantification with 1% FDR at precursor and protein levels)
- **Orbitrap** (Instrument series (e.g. QE HF-X) whose paired DDA/DIA acquisitions are processed)

## Evaluation signals

- DIA-NN output passes 1% FDR filtration at both peptide precursor and protein levels
- Ratio-of-change curves for DIA-based LC-MS/MS consensus QC metrics (e.g. MS2 intensity) show visibly earlier/higher excursions at maintenance-event timestamps than the DDA MS1-area curves
- Metric count matches the number of input files, with each file carrying instrument type and instrument ID assignments
- iDIA-QC produces both the metric matrices and the HTML report, with the reproduced curves matching the paper's finding that DIA metrics are more sensitive than DDA metrics around maintenance events

## Limitations

- Finding was established on mouse liver digest data from 21 mass spectrometers over 31 months; generalization to other QC and experimental samples requires further investigation
- The study could not cover all LC and MS instruments, especially newly emerging ones
- Analysis time per file is under 5 minutes per the README, but very large paired cohorts (e.g. 2754 DIA / 2638 DDA files) require batch processing

## Evidence

- [abstract] DIA-based LC-MS/MS-related consensus QC metrics exhibit higher sensitivity than DDA-based QC metrics in detecting changes in LC-MS status: "DIA-based LC-MS/MS-related consensus QC metrics exhibit higher sensitivity compared to DDA-based QC metrics in detecting changes in LC-MS status"
- [discussion] Conversion and DIA-NN analysis with 1% FDR: [evidence span withheld — anchor status: not_found]
- [discussion] 1% FDR at both peptide precursor and protein levels: "a data filtration criterion of 1% FDR at both peptide precursor and protein levels"
- [methods] File selection and instrument assignment in iDIA-QC: "Click Raw (in the Input pane), select your raw mass spectrometry data files."
- [readme] Supported raw formats and instrument type/ID assignment: "This software supports the .raw, .d, and .wiff raw data formats. Choose the type of instrument that generated the file. Assign an instrument ID to the corresponding instrument for this file."
- [readme] HTML report and metric matrix outputs: "There are two types of output files produced by iDIA-QC: html report, matrices (easy-to-use even without statistical software such as R/Python)."
