---
name: lc-ms-qualified-prediction
description: Use when a DIA-MS raw file (.raw/.d/.wiff) from Orbitrap, TripleTOF,
  or timsTOF instruments must be assessed for LC/MS run quality and receive an automated
  qualified/unqualified prediction.
when_to_use_negative:
- Input files are DDA or SRM acquisitions — the models were trained on DIA metrics;
  SRM-based QC tools 'cannot be directly applied to large scale quantitative proteomics
  experiments'
- The instrument is a newly emerging model outside the Orbitrap/TripleTOF/timsTOF
  series covered in the study
- The run has already been manually assessed and only archive-style reporting of existing
  metrics is needed
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: iDIA-QC
  role: Python GUI that orchestrates conversion, DIA-NN analysis, metric extraction,
    ML prediction, and HTML report generation
  repo: https://github.com/guomics-lab/iDIA-QC
- name: msConvert
  role: Converts vendor raw files (.d/.raw/.wiff) to mzXML for metric parsing
- name: DIA-NN
  role: v1.7.15 analysis of converted DIA files to derive peptide/protein quantification
    used for the remaining QC metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:86e59931acaba698e376f6b009b4665665ae18050b8a1d23a4d6258cffcdfc99
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
  - TripleTOF
  - iDIA-QC
  - msConvert
  - timsTOF Pro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lc-ms-qualified-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the iDIA-QC pipeline to convert multivendor DIA raw files (.raw/.d/.wiff) to mzXML via msConvert, analyze with DIA-NN (v1.7.15), extract 15 numerical QC metrics, and feed them to pretrained machine learning models that assign qualified/unqualified LC and MS performance labels with fault-cause guidance.

## When to use

Use when a DIA-MS raw file (.raw/.d/.wiff) from Orbitrap, TripleTOF, or timsTOF instruments must be assessed for LC/MS run quality and receive an automated qualified/unqualified prediction.

## When NOT to use

- Input files are DDA or SRM acquisitions — the models were trained on DIA metrics; SRM-based QC tools 'cannot be directly applied to large scale quantitative proteomics experiments'
- The instrument is a newly emerging model outside the Orbitrap/TripleTOF/timsTOF series covered in the study
- The run has already been manually assessed and only archive-style reporting of existing metrics is needed

## Inputs

- Vendor DIA raw files (.raw, .d, .wiff) from Orbitrap, TripleTOF, or TripleTOF-series instruments
- Instrument type designation for each file (Orbitrap / TripleTOF / timsTOF Pro)
- Instrument ID and notification settings (email or WeChat token ID from pushplus)

## Outputs

- Self-contained per-file HTML QC report with the extracted metrics and LC/MS qualified/unqualified predictions
- Flat metric matrices (usable without R/Python)
- Longitudinal quality summary, optionally shared via email or WeChat

## How to apply

Record the instrument type (Orbitrap, TripleTOF, or timsTOF Pro) that generated each raw file, then convert .d/.raw/.wiff files to mzXML using msConvert. Analyze the converted DIA files with DIA-NN (v1.7.15) under a 1% FDR filtration at both peptide precursor and protein levels to derive peptide and protein quantification. Extract the 15 numerical metrics by parsing the mzXML files and the DIA-NN outputs; remove features with >70% missing values and impute remaining missing values with the per-feature minimum. Feed the metric values into the pretrained machine learning model, which assigns a qualified/unqualified evaluation to LC and MS performance and provides potential instrument issues. Render the per-file result as a self-contained HTML QC report and write the metrics as flat matrices usable without statistical software such as R/Python.

## Related tools

- **iDIA-QC** (Python GUI that orchestrates conversion, DIA-NN analysis, metric extraction, ML prediction, and HTML report generation) — https://github.com/guomics-lab/iDIA-QC
- **msConvert** (Converts vendor raw files (.d/.raw/.wiff) to mzXML for metric parsing)
- **DIA-NN** (v1.7.15 analysis of converted DIA files to derive peptide/protein quantification used for the remaining QC metrics)

## Evaluation signals

- Every input raw file yields an HTML QC report containing the 15 extracted metrics plus LC and MS predictions
- Analysis time per file stays under 5 minutes (README performance principle)
- Output matrices load without statistical software such as R/Python
- Predicted LC/MS labels are comparable to expert manual assessment; reference performance is AUC 0.91 (LC) and 0.97 (MS) on the n=528 independent validation set
- Feature table after preprocessing contains no features with >70% missing values and no unimputed gaps

## Limitations

- The study 'is not possible to include all LC and MS instruments, particularly those emerging new instruments'
- Generalization of the workflow to other QC and experimental samples requires further investigation
- Model-building details are limited to a preprocessing/training/testing outline; hyperparameters and train/test split composition are not described

## Evidence
