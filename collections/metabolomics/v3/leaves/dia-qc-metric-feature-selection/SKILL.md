---
name: dia-qc-metric-feature-selection
description: 'Use when building or revising DIA-based QC classifiers from raw-file/DIA-NN-derived
  metrics: you have >140 candidate features spanning mzXML and DIA-NN outputs, and
  need a compact, non-redundant feature set with explicit missing-value handling before
  XGBoost training.'
when_to_use_negative:
- When the input is already a curated feature table with no missing values and a settled
  metric set — re-running selection would remove nothing and add no value.
- 'When the goal is DDA or SRM-based QC: the article notes SRM-based tools ''cannot
  be directly applied to large scale quantitative proteomics experiments'' and this
  feature-selection pipeline is built specifically for DIA raw files analyzed by DIA-NN.'
- When working with newly emerging LC/MS instruments not represented in the 21-instrument
  training corpus — the study itself notes it 'is not possible to include all LC and
  MS instruments, particularly those emerging new instruments', so the 15-metric subset
  may not transfer.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: Standalone Python GUI that extracts the 15 QC metrics from raw files and DIA-NN
    results and applies the pre-trained machine-learning models after feature selection
    is complete
  repo: github.com/guomics-lab/iDIA-QC
- name: msConvert
  role: Converts .d/.raw/.wiff vendor raw files to mzXML format, which is the input
    from which many raw-file QC metrics are computed
- name: DIA-NN
  role: Derives peptide and protein quantification results from the DIA files, supplying
    the precursor-level features (F2, F13, F14 and peptide precursor metrics) used
    for feature selection
- name: XGBoost
  role: Gradient-boosted classifier trained with five-fold cross-validation on the
    selected and imputed feature matrices to yield LC and MS QC models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:dbbdf747229d0edfbf6e4e7f4d32a67ae291af2209dbb2d86f71e5bf73ee0272
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
  - XGBoost
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dia-qc-metric-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curate and reduce the large pool of candidate DIA LC-MS quality-control metrics down to 15 practically applicable features, dropping any feature missing in >70% of files and imputing remaining NAs with per-feature minima, so that downstream machine-learning QC classifiers (LC and MS models) train on a clean, informative feature matrix.

## When to use

Use when building or revising DIA-based QC classifiers from raw-file/DIA-NN-derived metrics: you have >140 candidate features spanning mzXML and DIA-NN outputs, and need a compact, non-redundant feature set with explicit missing-value handling before XGBoost training.

## When NOT to use

- When the input is already a curated feature table with no missing values and a settled metric set — re-running selection would remove nothing and add no value.
- When the goal is DDA or SRM-based QC: the article notes SRM-based tools 'cannot be directly applied to large scale quantitative proteomics experiments' and this feature-selection pipeline is built specifically for DIA raw files analyzed by DIA-NN.
- When working with newly emerging LC/MS instruments not represented in the 21-instrument training corpus — the study itself notes it 'is not possible to include all LC and MS instruments, particularly those emerging new instruments', so the 15-metric subset may not transfer.

## Inputs

- mzXML files converted by msConvert from .d/.raw/.wiff vendor raw formats
- DIA-NN (v1.7.15) peptide and protein quantification results for each DIA file
- Expert-inspected qualified/unqualified QC labels for the 2110 training DIA files
- Candidate metric pool of >140 LC-MS/MS QC metrics (precursor-level and raw-file features)

## Outputs

- Filtered and imputed feature matrix per DIA file (features with >70% NA removed; remaining NAs imputed with per-feature minimum)
- 15 practically applicable QC metrics split into LC (F2, F13, F14 + 19 precursor metrics) and MS (F1, F3–F12 + 14 precursor metrics) subsets
- Clean feature table ready for five-fold cross-validated XGBoost classifier training

## How to apply

First extract >140 candidate QC metrics from each DIA raw file using its mzXML (converted from .d/.raw/.wiff by msConvert) and DIA-NN (v1.7.15) peptide/protein quantification results. Through expert review, narrow the pool to 15 practically applicable metrics, partitioned as F1, F3–F12 from DIA raw files and F2, F13, F14 from extracted precursor-related features. Remove features with missing rates >70% NA, then impute remaining missing values with the minimum value of each respective feature. Decide whether a metric belongs in the LC model (19 peptide precursor metrics plus F3) or the MS model (14 peptide precursor metrics plus F4/F5/F6/F7/F11/F12), since these disjoint subsets drive the two separate XGBoost classifiers. The rationale is that only raw-file/DIA-NN-derived features with high coverage and expert-validated informativeness yield classifiers whose blinded validation AUCs (0.91 LC / 0.97 MS on n=528; 0.78 LC / 0.94 MS on n=116) can be reproduced.

## Related tools

- **iDIA-QC** (Standalone Python GUI that extracts the 15 QC metrics from raw files and DIA-NN results and applies the pre-trained machine-learning models after feature selection is complete) — github.com/guomics-lab/iDIA-QC
- **msConvert** (Converts .d/.raw/.wiff vendor raw files to mzXML format, which is the input from which many raw-file QC metrics are computed)
- **DIA-NN** (Derives peptide and protein quantification results from the DIA files, supplying the precursor-level features (F2, F13, F14 and peptide precursor metrics) used for feature selection)
- **XGBoost** (Gradient-boosted classifier trained with five-fold cross-validation on the selected and imputed feature matrices to yield LC and MS QC models)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('metrics.csv'); keep = df.columns[df.isna().mean() <= 0.70]; df[keep] = df[keep].fillna(df[keep].min())"
```

## Evaluation signals

- Confirm no retained feature has missing rate >70% NA in the training cohort; check imputed values equal each feature's own minimum before scaling or splitting.
- Verify the final metric count is 15 and that the LC and MS feature sets map exactly to the paper's assignment: LC = F2, F13, F14 + 19 peptide precursor metrics; MS = F1, F3–F12 + 14 peptide precursor metrics.
- Cross-validate XGBoost on the 2110 training files with five folds and confirm the trained LC and MS models reproduce the reported blinded AUCs (0.91 LC / 0.97 MS on validation set 1, n=528; 0.78 LC / 0.94 MS on validation set 2, n=116 QE HF files).
- Ensure that at the 1% FDR cutoff applied at both peptide precursor and protein levels, DIA-NN outputs are usable as feature sources without additional filtering beyond the missing-value rule.
- Longitudinal QC reports generated downstream should show per-file qualified/unqualified status consistent with expert labels on a held-out subset, indicating the selected features carry sufficient discriminative signal.

## Limitations

- Hyperparameters of the XGBoost models and the exact train/test split composition are not described in the article, so re-implementation may require tuning to match reported AUCs.
- The 15-metric subset was curated from data spanning 21 mass spectrometers across nine laboratories over 31 months; instruments or sample types outside that space may not be well represented.
- Features with high missing rates (>70% NA) are dropped wholesale, which can silently discard informative-but-sparsely-recorded metrics on newer instrument configurations.
- The article notes no changelog and that generalization of the workflow to other QC and experimental samples requires further investigation.

## Evidence
