---
name: xgboost-gradient-boosted-classifier-training
description: Use when you have expert-labelled DIA-MS files with numeric QC features
  (precursor metrics like F2/F13/F14 plus DIA raw-file features F1, F3–F12) and need
  a blinded binary qualified/unqualified LC or MS performance classifier.
when_to_use_negative:
- Inputs are already-qualified QC calls or binary prediction labels rather than per-file
  numeric QC features — use a reporting/visualisation skill instead
- 'Data is DDA or SRM rather than DIA-MS: the paper states prior DDA/SRM QC pipelines
  cannot be directly applied to large-scale DIA experiments'
- Instrument type is not represented in the training instruments (Orbitrap, TripleTOF,
  timsTOF Pro series) — the article notes newly emerging instruments were not covered
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: iDIA-QC
  role: Python GUI that converts vendor raw files (.raw/.d/.wiff) to mzXML via msConvert,
    runs DIA-NN, extracts the 15 QC metrics, trains the XGBoost LC/MS classifiers
    with five-fold CV, and reports predictions via HTML/matrices/email/WeChat
  repo: github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: Analyzes each DIA file (v1.7.15) to derive peptide and protein quantification
    from which peptide precursor QC metrics are extracted for model input
- name: msConvert
  role: Converts vendor raw files (.d/.raw/.wiff) to mzXML format from which raw-file-based
    features (F1, F3–F12) are extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:e283049dac0ce2a441250c677634260fb4830539a04768b7e03dd32b8a1afd61
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
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# xgboost-gradient-boosted-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train XGBoost gradient-boosted classifiers that predict qualified versus unqualified LC and MS performance from DIA-MS QC metrics, as in iDIA-QC's five-fold cross-validated models achieving blind-validation AUCs of 0.91/0.97 (LC/MS, validation set 1) and 0.78/0.94 (validation set 2).

## When to use

Use when you have expert-labelled DIA-MS files with numeric QC features (precursor metrics like F2/F13/F14 plus DIA raw-file features F1, F3–F12) and need a blinded binary qualified/unqualified LC or MS performance classifier.

## When NOT to use

- Inputs are already-qualified QC calls or binary prediction labels rather than per-file numeric QC features — use a reporting/visualisation skill instead
- Data is DDA or SRM rather than DIA-MS: the paper states prior DDA/SRM QC pipelines cannot be directly applied to large-scale DIA experiments
- Instrument type is not represented in the training instruments (Orbitrap, TripleTOF, timsTOF Pro series) — the article notes newly emerging instruments were not covered

## Inputs

- Expert-inspected QC metric table for DIA raw files (one row per file, with 15 prioritized metrics: peptide precursor metrics and DIA raw file-based features F1–F14) plus qualified/unqualified expert labels
- Training set of DIA-MS files with labels (iDIA-QC: 2110 files); independent blinded validation sets (e.g. 528 files; 116 QE HF files)

## Outputs

- Two fitted XGBoost model files (LC and MS QC classifiers)
- Validation results table with blind-prediction ROC AUCs for LC and MS (expected: 0.91/0.97 on n=528; 0.78/0.94 on n=116)
- Per-file qualified/unqualified predictions and feature importance/status matrices for longitudinal QC reporting

## How to apply

Start from an expert-inspected QC metric table (iDIA-QC: 2110 training DIA files labelled qualified/unqualified by 21 experts), drop features with >70% NA and impute remaining missing values with each feature's minimum. Assemble the LC feature matrix (19 peptide precursor metrics plus F3) and the MS feature matrix (14 peptide precursor metrics plus F4/F5/F6/F7/F11/F12) against the qualified/unqualified labels. Train each gradient-boosted classifier (XGBoost) with five-fold cross-validation on the training set to select the final LC and MS QC models. Then apply the fitted models blinded to independent validation DIA files (e.g. n=528 and n=116 QE HF files) and report ROC AUC; reproduce paper targets (0.91/0.97 and 0.78/0.94) to confirm correct application. Save the fitted model objects plus a validation-AUC results table for downstream longitudinal QC reporting (iDIA-QC emits HTML reports and feature/status matrices).

## Related tools

- **iDIA-QC** (Python GUI that converts vendor raw files (.raw/.d/.wiff) to mzXML via msConvert, runs DIA-NN, extracts the 15 QC metrics, trains the XGBoost LC/MS classifiers with five-fold CV, and reports predictions via HTML/matrices/email/WeChat) — github.com/guomics-lab/iDIA-QC
- **DIA-NN** (Analyzes each DIA file (v1.7.15) to derive peptide and protein quantification from which peptide precursor QC metrics are extracted for model input)
- **msConvert** (Converts vendor raw files (.d/.raw/.wiff) to mzXML format from which raw-file-based features (F1, F3–F12) are extracted)

## Examples

```
python -c "import iDIA_QC; iDIA_QC.run(raw_files=raws, instrument='Orbitrap', output_dir='out/')"  # iDIA-QC GUI: click Raw, select .raw/.d/.wiff files, choose instrument type, click Run
```

## Evaluation signals

- Blind validation AUC on independent set 1 (n=528) matches 0.91 for LC and 0.97 for MS
- Blind validation AUC on set 2 (n=116 QE HF files) matches 0.78 (LC) and 0.94 (MS)
- Feature-missingness filter applied: features with >70% NA removed, remaining NAs imputed with per-feature minimum
- Five-fold cross-validation used for model selection; both LC (19 precursor metrics + F3) and MS (14 precursor metrics + F4/F5/F6/F7/F11/F12) feature matrices assembled
- Per-file runtime under ~5 minutes and outputs include HTML report plus feature/status matrices as specified by iDIA-QC

## Limitations

- QC metrics were narrowed from over 140 candidate metrics to 15 practically applicable ones; other metrics may not be supported
- Study data covers 21 mass spectrometers (Orbitrap, TripleTOF, timsTOF Pro series) over 31 months; newly emerging instrument types are not covered
- Generalization of the workflow to other QC samples and experimental settings requires further investigation
- High-missingness features (>70% NA) are discarded, and remaining missing values are imputed with per-feature minimums, which may bias borderline files

## Evidence
