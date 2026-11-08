---
name: five-fold-cross-validation-model-selection
description: Use when a labelled DIA QC metric table (e.g. 2110 expert-inspected files
  from 21 mass spectrometers) must yield robust LC/MS qualified-vs-unqualified classifiers
  whose generalisation to blinded validation files can be measured by ROC AUC.
when_to_use_negative:
- Input is unlabelled DIA data with no expert QC annotations — cross-validated classification
  of qualified/unqualified status cannot be performed
- The goal is QC of DDA or SRM data, which the article notes cannot be directly applied
  to large-scale DIA experiments and vice versa
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0121
tools:
- name: iDIA-QC
  role: Python GUI that extracts the 15 QC metrics from DIA files and feeds them into
    the machine-learning classifier used for LC/MS QC prediction
  repo: https://github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:3663b5e25ae461a289205d762e0e1e8bfe517f59acbedc0a9c1d97c7aff69e4d
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
  - iDIA-QC
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# five-fold-cross-validation-model-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Select LC and MS quality-control classifier models by training gradient-boosted models (XGBoost) with five-fold cross-validation on expert-labelled DIA files, then evaluating them blinded on independent validation sets.

## When to use

Use when a labelled DIA QC metric table (e.g. 2110 expert-inspected files from 21 mass spectrometers) must yield robust LC/MS qualified-vs-unqualified classifiers whose generalisation to blinded validation files can be measured by ROC AUC.

## When NOT to use

- Input is unlabelled DIA data with no expert QC annotations — cross-validated classification of qualified/unqualified status cannot be performed
- The goal is QC of DDA or SRM data, which the article notes cannot be directly applied to large-scale DIA experiments and vice versa

## Inputs

- Expert-inspected QC metric table for 2110 DIA files (features F1-F14 derived from mzXML and DIA-NN results) with qualified/unqualified LC and MS labels
- Independent blinded validation DIA datasets (n=528 and n=116 QE HF files)

## Outputs

- Fitted LC XGBoost model file
- Fitted MS XGBoost model file
- Results table of validation ROC AUCs (LC and MS per validation set)

## How to apply

Assemble feature matrices from the 15 DIA QC metrics extracted from mzXML and DIA-NN outputs: the LC model uses 19 peptide precursor metrics plus F3, and the MS model uses 14 peptide precursor metrics plus F4/F5/F6/F7/F11/F12. Remove features with >70% missing values and impute remaining missing values with the minimum of each respective feature. Train XGBoost classifiers with five-fold cross-validation on the 2110-file training set to select the LC and MS QC models. Apply the trained models, blinded, to independent validation sets (n=528 and n=116 QE HF files) and compute ROC AUC. Report the fitted model files and a results table of validation AUCs (expected LC 0.91/0.78 and MS 0.97/0.94).

## Related tools

- **iDIA-QC** (Python GUI that extracts the 15 QC metrics from DIA files and feeds them into the machine-learning classifier used for LC/MS QC prediction) — https://github.com/guomics-lab/iDIA-QC

## Examples

```
python -m xgboost --config train_config.json --data metrics_2110.csv --folds 5 --output lc_model.json ms_model.json
```

## Evaluation signals

- Validation AUC on the first blinded set must reach ~0.91 for LC and ~0.97 for MS
- Validation AUC on the second QE HF set must reach ~0.78 for LC and ~0.94 for MS
- Features with >70% missing values must be removed before training; remaining NAs imputed with per-feature minimum
- Both peptide precursor and protein FDR filtration at 1% is applied to the underlying DIA-NN quantification
- Evaluation is blind: no labels from validation files used during training or model selection

## Limitations

- Hyperparameters and exact train/test split composition are not described in the article's model-building outline
- The workflow was validated only on specific instruments (Orbitrap, TripleTOF, timsTOF Pro); emerging instruments were not covered
- Generalisation to other QC or experimental samples requires further investigation

## Evidence
