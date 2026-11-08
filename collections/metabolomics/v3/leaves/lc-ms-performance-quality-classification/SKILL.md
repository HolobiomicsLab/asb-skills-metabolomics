---
name: lc-ms-performance-quality-classification
description: Use when evaluating whether DIA raw files (from Orbitrap, TripleTOF,
  or timsTOF series instruments) show degraded LC or MS performance, or when binary
  qualified/unqualified labels must be predicted from extracted QC metrics rather
  than inspected manually.
when_to_use_negative:
- Input consists of DDA or SRM raw files rather than DIA acquisitions — the classifiers
  are trained on DIA-specific precursor metrics and cannot be applied directly to
  large-scale DDA or SRM experiments
- The instrument type is outside the Orbitrap, TripleTOF, or timsTOF series scope
  the 15 metrics were designed for, as the article notes newly emerging instruments
  were not covered
- A single QC metric needs thresholding rather than a combined classifier — this skill
  is for multivariate qualified/unqualified prediction, not per-metric QC rules
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: iDIA-QC
  role: Standalone GUI that converts raw DIA files, runs DIA-NN, extracts the 15 QC
    metrics, and hosts the trained XGBoost LC/MS classifiers
  repo: https://github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: Provides peptide and protein quantification results (with 1% FDR filtering)
    that feed the precursor-level QC metrics used for classification
- name: msConvert
  role: Converts vendor .d/.raw/.wiff raw files to mzXML from which numerical QC metrics
    are extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:c4ddd49531ac19ebf6ba6d8c1993b6c1a9c20784e67c9ef630fa6cc3e81a6085
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
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lc-ms-performance-quality-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains and applies XGBoost gradient-boosted classifiers to predict whether LC and MS subsystems of a DIA acquisition were qualified or unqualified, using 15 precursor- and raw-file-level metrics extracted by iDIA-QC and expert labels, reproducing validation AUCs of 0.91 (LC) / 0.97 (MS) on a 528-file blinded set and 0.78 (LC) / 0.94 (MS) on a 116-file set.

## When to use

Use when evaluating whether DIA raw files (from Orbitrap, TripleTOF, or timsTOF series instruments) show degraded LC or MS performance, or when binary qualified/unqualified labels must be predicted from extracted QC metrics rather than inspected manually.

## When NOT to use

- Input consists of DDA or SRM raw files rather than DIA acquisitions — the classifiers are trained on DIA-specific precursor metrics and cannot be applied directly to large-scale DDA or SRM experiments
- The instrument type is outside the Orbitrap, TripleTOF, or timsTOF series scope the 15 metrics were designed for, as the article notes newly emerging instruments were not covered
- A single QC metric needs thresholding rather than a combined classifier — this skill is for multivariate qualified/unqualified prediction, not per-metric QC rules

## Inputs

- DIA raw files in .raw, .d, or .wiff format (vendor format depends on Orbitrap, TripleTOF, or timsTOF series instrument)
- mzXML files converted from raw vendor files via msConvert
- DIA-NN v1.7.15 peptide/protein quantification results (1% FDR at both peptide precursor and protein levels)
- Expert-inspected QC metric table for the training set (n=2110) with qualified/unqualified labels for LC and MS

## Outputs

- Two fitted XGBoost model files (LC model and MS model)
- Predicted qualified/unqualified status per DIA file for LC and MS subsystems
- ROC AUC values on blinded validation sets (target ~0.91 LC / 0.97 MS for set 1; ~0.78 LC / 0.94 MS for set 2)
- Results table of validation AUCs

## How to apply

Start from the 15 QC metrics iDIA-QC extracts per file: the LC model uses 19 peptide precursor metrics plus F3, while the MS model uses 14 peptide precursor metrics plus F4/F5/F6/F7/F11/F12. Before training, remove features with >70% missing values and impute remaining NAs with each feature's minimum; apply 1% FDR at both peptide precursor and protein levels when building the DIA-NN quantification inputs. Train each XGBoost classifier with five-fold cross-validation on the 2110-file training set carrying expert qualified/unqualified labels, then apply the fitted models blindly to validation DIA files to obtain predicted probabilities and ROC AUCs (expect ~0.91 LC / 0.97 MS on the 528-file set and ~0.78 LC / 0.94 MS on the 116-file QE HF subset). Save both model files and a results table of validation AUCs.

## Related tools

- **iDIA-QC** (Standalone GUI that converts raw DIA files, runs DIA-NN, extracts the 15 QC metrics, and hosts the trained XGBoost LC/MS classifiers) — https://github.com/guomics-lab/iDIA-QC
- **DIA-NN** (Provides peptide and protein quantification results (with 1% FDR filtering) that feed the precursor-level QC metrics used for classification)
- **msConvert** (Converts vendor .d/.raw/.wiff raw files to mzXML from which numerical QC metrics are extracted)

## Examples

```
Launch iDIA-QC GUI: Click Raw → select .raw/.d/.wiff files → choose instrument type → assign instrument ID → set output directory (html report + matrices) → Run
```

## Evaluation signals

- Validation set 1 (n=528 blinded DIA files) yields ROC AUC ≈0.91 for LC and ≈0.97 for MS
- Validation set 2 (n=116 QE HF files) yields ROC AUC ≈0.78 for LC and ≈0.94 for MS
- Feature table after preprocessing contains no columns with >70% missing values, and remaining missing entries equal the per-feature minimum
- Per-file analysis completes in under 5 minutes (README speed benchmark)
- Output includes both the two fitted model files and a results table with the validation AUCs

## Limitations

- Not validated on instruments outside the Orbitrap, TripleTOF, and timsTOF series, especially newly emerging instruments
- Generalization to other QC and experimental samples requires further investigation
- Model-building details such as hyperparameters and train/test split composition are not described in the article text
- Per-file processing must remain under 5 minutes for the pipeline to stay practical, so very large file sets may need batching

## Evidence
