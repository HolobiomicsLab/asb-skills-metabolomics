---
name: diann-quantification-output-parsing
description: Use when raw vendor DIA files (.raw/.d/.wiff) from Orbitrap, TripleTOF,
  or timsTOF Pro instruments must be converted to mzXML, analyzed with DIA-NN, and
  parsed into the QC metric set feeding iDIA-QC's LC/MS quality predictions.
when_to_use_negative:
- Input is DDA data — the article notes existing QC pipelines for DDA/SRM cannot be
  directly applied to large-scale DIA experiments, and this skill targets DIA-NN DIA
  analysis outputs
- The instrument generating the files is outside the Orbitrap, TripleTOF, or timsTOF
  Pro series — the article states it is not possible to include all LC and MS instruments,
  particularly emerging new ones
- Only raw metric tables are needed with no quality prediction — use a direct metric
  extractor rather than routing through the DIA-NN-based iDIA-QC pipeline
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: DIA-NN
  role: Analyze converted DIA files to derive peptide and protein quantification results
    from which most of the QC metrics are extracted
- name: msConvert
  role: Convert .d/.raw/.wiff vendor raw files to mzXML format prior to metric extraction
- name: iDIA-QC
  role: Python GUI that wraps DIA-NN and msConvert, extracts the 15 metrics, and runs
    machine-learning quality prediction
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:775dccf915f61a1fa923f266ae08bb9acaecc1050e310598bea11fa64c9484f3
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
  - TripleTOF
  - iDIA-QC
  - msConvert
  - timsTOF Pro
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# diann-quantification-output-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse DIA-NN (v1.7.15) analysis outputs to extract the numerical QC metrics underlying iDIA-QC's per-file quality evaluation, alongside mzXML-parsed (F3, F5) and timsTOF MGF-parsed (F15) metrics.

## When to use

Use when raw vendor DIA files (.raw/.d/.wiff) from Orbitrap, TripleTOF, or timsTOF Pro instruments must be converted to mzXML, analyzed with DIA-NN, and parsed into the QC metric set feeding iDIA-QC's LC/MS quality predictions.

## When NOT to use

- Input is DDA data — the article notes existing QC pipelines for DDA/SRM cannot be directly applied to large-scale DIA experiments, and this skill targets DIA-NN DIA analysis outputs
- The instrument generating the files is outside the Orbitrap, TripleTOF, or timsTOF Pro series — the article states it is not possible to include all LC and MS instruments, particularly emerging new ones
- Only raw metric tables are needed with no quality prediction — use a direct metric extractor rather than routing through the DIA-NN-based iDIA-QC pipeline

## Inputs

- Vendor raw DIA files (.raw, .d, .wiff) from Orbitrap, TripleTOF, or timsTOF Pro instruments
- mzXML files produced by msConvert conversion
- DIA-NN (v1.7.15) peptide and protein quantification outputs
- MGF files acquired with the timsTOF Pro instrument (for F15)

## Outputs

- 17 numerical QC metrics extracted per DIA raw file
- Qualified/unqualified LC and MS performance evaluations with fault-cause guidance
- Self-contained HTML QC report per file
- Flat metric matrices usable without statistical software such as R/Python

## How to apply

First convert each vendor raw file (.raw/.d/.wiff) to mzXML using msConvert, then analyze the converted DIA files with DIA-NN (v1.7.15) under a 1% FDR criterion at both peptide precursor and protein levels to derive peptide and protein quantification results. Parse the DIA-NN outputs for the majority of the 17 numerical QC metrics, parsing mzXML files for metrics F3 and F5, and MGF files from timsTOF Pro acquisition for F15. Preprocess features by removing those with >70% missing values and imputing remaining missing values with the per-feature minimum. Feed the extracted metric values into the pretrained machine learning model, which assigns a qualified/unqualified evaluation to LC and MS performance and generates fault-cause guidance.

## Related tools

- **DIA-NN** (Analyze converted DIA files to derive peptide and protein quantification results from which most of the QC metrics are extracted)
- **msConvert** (Convert .d/.raw/.wiff vendor raw files to mzXML format prior to metric extraction)
- **iDIA-QC** (Python GUI that wraps DIA-NN and msConvert, extracts the 15 metrics, and runs machine-learning quality prediction) — github.com/guomics-lab/iDIA-QC

## Evaluation signals

- Each input raw file yields an HTML QC report containing all 17 metrics plus LC/MS predictions
- Analysis completes with per-file runtime under 5 minutes
- DIA-NN analysis applied 1% FDR filtration at both peptide precursor and protein levels
- Features with >70% missing values were removed and other missing values imputed by the per-feature minimum
- Output matrices are usable without statistical software such as R/Python

## Limitations

- The study could not cover all LC and MS instruments, especially newly emerging ones
- Generalization of the workflow to other QC and experimental samples requires further investigation
- Model-building details are limited to a preprocessing/training/testing outline; hyperparameters and train/test split composition are not described

## Evidence
