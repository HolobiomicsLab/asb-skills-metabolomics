---
name: missing-value-filtering-and-imputation
description: Use when preparing expert-inspected or DIA-NN-derived QC feature tables
  (with many NA entries across DIA files) for machine learning, before training the
  LC/MS qualified-vs-unqualified classifiers.
when_to_use_negative:
- Input data already lacks missing values or the missing rate is far below the 70%
  threshold, so filtering would remove no features and imputation is unnecessary
- Applying the preprocessing to non-DIA data types (e.g. DDA or SRM QC data) where
  the >70% cutoff and minimum imputation were not validated
edam_operation: http://edamontology.org/operation_3590
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: Python GUI tool whose ML pipeline performs the >70% missing-value filter and
    minimum-value imputation on the 15 DIA QC metrics before model training
  repo: github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: Upstream analysis that produces the peptide/protein quantification results
    from which the numerical QC metrics with NA values are extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:1821c8949d539b1557aa48bab2688307e0d6f2048a14fe586a48e25f523d162a
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
  edam_operation: http://edamontology.org/operation_3590
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Missing-value filtering and imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing step for DIA QC feature matrices in which features missing in >70% of files are removed and remaining NA values are imputed with each feature's minimum observed value, preparing the 2110-file training data for XGBoost-based LC and MS quality classifiers.

## When to use

Use when preparing expert-inspected or DIA-NN-derived QC feature tables (with many NA entries across DIA files) for machine learning, before training the LC/MS qualified-vs-unqualified classifiers.

## When NOT to use

- Input data already lacks missing values or the missing rate is far below the 70% threshold, so filtering would remove no features and imputation is unnecessary
- Applying the preprocessing to non-DIA data types (e.g. DDA or SRM QC data) where the >70% cutoff and minimum imputation were not validated

## Inputs

- QC metric table with numerical features extracted per DIA raw file (mzXML- and DIA-NN-derived metrics, e.g. F1-F14)
- Expert-inspected qualified/unqualified labels for the training DIA files (n=2110)

## Outputs

- Filtered and minimum-imputed feature matrix ready for XGBoost training
- List of dropped features (those with >70% missing values)

## How to apply

Assemble the feature matrix from numerical metrics extracted per DIA raw file (peptide precursor metrics plus raw-file metrics such as F1, F3-F14), then scan each feature column for its missing-value rate and drop any feature with >70% NA. For surviving features, impute each missing entry with the minimum value observed for that specific feature, a low-signal substitution consistent with iDIA-QC's training setup on 2110 files. The imputed matrix then feeds five-fold cross-validated XGBoost training of separate LC and MS quality models. Rationale: high-missingness metrics carry little discriminative signal, and minimum imputation keeps unquantified low performers in a plausible low range rather than distorting the distribution with a mean.

## Related tools

- **iDIA-QC** (Python GUI tool whose ML pipeline performs the >70% missing-value filter and minimum-value imputation on the 15 DIA QC metrics before model training) — github.com/guomics-lab/iDIA-QC
- **DIA-NN** (Upstream analysis that produces the peptide/protein quantification results from which the numerical QC metrics with NA values are extracted)

## Examples

```
python idiaqc --input metrics_table.csv --output imputed_matrix.csv --missing-threshold 0.70 --impute-min
```

## Evaluation signals

- No feature in the output matrix has a missing rate above 70%; the count of dropped features matches the count of columns with >70% NA in the input
- Every remaining NA in the matrix equals the per-feature minimum value of that column (spot-check several imputed entries)
- The imputed matrix has the same number of rows as the input (2110 training files) and retains only the features that survived filtering
- Downstream XGBoost cross-validation and blinded validation AUCs (0.91 LC / 0.97 MS on n=528; 0.78 LC / 0.94 MS on n=116) are reproducible with the preprocessed data

## Limitations

- The >70% missing cutoff and minimum-imputation choice were tuned for DIA QC data from timsTOF, TripleTOF, and Orbitrap instruments and may not generalize to other data types
- Minimum imputation can bias low-end values; if a feature's minimum is an outlier, the imputed entries become unrealistic low signals
- The article notes that generalization of the full workflow to other QC and experimental samples requires further investigation

## Evidence
