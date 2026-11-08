---
name: zero-variance-feature-filtering
description: Use when preparing CCLE/DepMap-derived omics feature matrices (or any
  cell-line-by-feature table) as input to Metab8D random forest models and you need
  to drop features carrying no cross-sample variation before Z-scoring and 5-fold
  CV training.
when_to_use_negative:
- The input table has already been variance-filtered or is already a normalized (Z-scored)
  feature matrix — re-filtering adds nothing and pre-normalizing would leak information
  across CV folds
- You are working with single-sample or single-cell data where per-feature cross-sample
  variance is not the criterion (Metab8D filtering assumes a panel of ~1000 matched
  cell lines)
- Binary mutation matrices destined for exclusion — Metab8D excluded mutations from
  random forest results due to their binary structure, so variance filtering there
  serves only the reported count check
tools:
- name: Python
  role: Scripting language used to load, impute (NaN->0), and variance-filter the
    CCLE omics matrices
- name: scikit-learn
  role: Underlying ML library for the Metab8D random forests and downstream Z-score
    normalization within CV folds
- name: BioTools.fr
  role: Web service used to convert RNA splicing ensemble IDs to common gene names
    during the same preprocessing pass
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:7c66a3886a1f4a2245b6ba4151c841939ca8e4dd6e918716b0bdeda305471d31
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BioTools.fr
  - Python
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# zero-variance-feature-filtering

## Summary

A preprocessing step from the Metab8D/Recon8D pipeline that removes features with zero variance from raw CCLE omics feature matrices (e.g., copy number, mutations, DNA methylation, histone PTMs, transcriptomics, splicing, miRNA, lncRNA, proteomics, phosphoproteomics), since features identical across all ~1000 cancer cell lines carry no predictive signal for the metabolite random forest models.

## When to use

Use when preparing CCLE/DepMap-derived omics feature matrices (or any cell-line-by-feature table) as input to Metab8D random forest models and you need to drop features carrying no cross-sample variation before Z-scoring and 5-fold CV training.

## When NOT to use

- The input table has already been variance-filtered or is already a normalized (Z-scored) feature matrix — re-filtering adds nothing and pre-normalizing would leak information across CV folds
- You are working with single-sample or single-cell data where per-feature cross-sample variance is not the criterion (Metab8D filtering assumes a panel of ~1000 matched cell lines)
- Binary mutation matrices destined for exclusion — Metab8D excluded mutations from random forest results due to their binary structure, so variance filtering there serves only the reported count check

## Inputs

- Raw CCLE/DepMap omics feature matrices per biomolecular class (CNV, mutations, DNA methylation, histone PTMs, coding transcripts, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics)
- Missing-value-imputed matrices (NaNs already replaced with zeros, repeated observations collapsed by median)

## Outputs

- Filtered per-class feature matrices with zero-variance columns removed
- Recorded feature counts per omics class (e.g., 3,767 lncRNA, 2,957 splicing, 17,203 mutation features) in preprocessed_features.csv

## How to apply

Load the raw per-biomolecular-class CCLE feature matrices and first replace missing values (NaN) with zeros per the Metab8D convention, collapsing repeated observations of a feature by taking the median. Then compute the variance of each feature column across cell lines and drop every feature with 0 variance, which carries no predictive signal for downstream modeling. Feature-name standardization (e.g., converting RNA splicing ensemble IDs to common names via BioTools.fr) is done in the same pass so that filtered features remain interpretable. Crucially, do NOT Z-score before filtering; Z-score normalization is applied later within each 5-fold CV training split to avoid information leakage. Verify the surviving feature counts per class against reported values (e.g., 3,767 lncRNAs, 2,957 splicing events, 17,203 mutation features) before saving the matrices.

## Related tools

- **Python** (Scripting language used to load, impute (NaN->0), and variance-filter the CCLE omics matrices)
- **scikit-learn** (Underlying ML library for the Metab8D random forests and downstream Z-score normalization within CV folds)
- **BioTools.fr** (Web service used to convert RNA splicing ensemble IDs to common gene names during the same preprocessing pass)

## Examples

```
df = df.fillna(0); df = df.loc[:, df.var(axis=0) > 0]
```

## Evaluation signals

- Every surviving feature column in each omics-class matrix has non-zero variance across the ~1000 CCLE cell lines
- Resulting feature counts match the reported values (3,767 lncRNAs, 2,957 splicing events, 17,203 mutation features, etc.)
- No NaNs remain in the saved matrices (they were replaced with zeros prior to filtering)
- Preprocessed per-class matrices are saved to preprocessed_features.csv with feature counts recorded alongside
- Z-score normalization is confirmed to occur within each 5-fold CV training split, not on the full pre-filtering matrix

## Limitations

- Zero-variance filtering is only meaningful given sufficient samples; the article notes omics with too few samples (cell lines) may not perform as well
- Features near-zero (but not exactly zero) variance may still contribute noise; Metab8D only explicitly removes exactly 0-variance features
- The preprocessing convention (NaN -> zero imputation) is a design choice of the pipeline and the repository separately provides KNN imputation examples in preprocessing_example.ipynb

## Evidence

- [discussion] zero-variance carries no signal: [evidence span withheld — anchor status: not_found]
