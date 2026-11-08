---
name: multiblock-pls-regression-modeling
description: Use when you have two or more sample-aligned LC-MS feature-intensity
  tables (e.g. HPOS/LPOS/LNEG or multi-assay serum assays) and a numeric outcome (continuous
  bilirubin levels or encoded biological sex) to predict jointly across blocks.
when_to_use_negative:
- You have only a single assay block and no interblock structure to model — a single-block
  PLS or standard regression suffices
- The assay blocks are not sample-aligned (features from different assays cannot be
  indexed onto a common sample order), which breaks block-wise NIPALS fitting
- The goal is annotation or structural clustering of already-selected features rather
  than outcome modeling — use MamsiStructSearch instead
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: mamsi
  role: Provides MamsiPls, which extends mbpls with NIPALS MB-PLS fitting, estimate_lv,
    evaluate_class_model/evaluate_regression_model, montecarlo_cv, and block_importance
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mbpls 1.0.4
  role: Pinned dependency implementing the underlying multiblock PLS algorithm
  repo: https://github.com/kopeckylukas/py-mamsi
- name: scikit-learn
  role: train_test_split for sample-aligned 90:10 train/test splitting of all assay
    blocks
- name: pandas
  role: Loading assay feature tables and metadata, prefixing columns, and concatenating
    blocks with pd.concat(axis=1)
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:1938e77665fa033aaf3e489cee96c6405e0e7f5d0f971dfc0e558b30277dc8fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - mamsi
  - matplotlib
  - mbpls
  - mbpls 1.0.4
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multiblock-pls-regression-modeling

## Summary

Fit a multi-block partial least-squares (MB-PLS) model to integrate multiple LC-MS metabolomics assay blocks against a continuous or binary phenotypic outcome, selecting latent variables by cross-validation (estimate_lv) and evaluating performance with k-fold or Monte Carlo cross-validation. This is the core modeling step of the MAMSI workflow and outperforms single-assay models by modeling interblock relationships.

## When to use

Use when you have two or more sample-aligned LC-MS feature-intensity tables (e.g. HPOS/LPOS/LNEG or multi-assay serum assays) and a numeric outcome (continuous bilirubin levels or encoded biological sex) to predict jointly across blocks.

## When NOT to use

- You have only a single assay block and no interblock structure to model — a single-block PLS or standard regression suffices
- The assay blocks are not sample-aligned (features from different assays cannot be indexed onto a common sample order), which breaks block-wise NIPALS fitting
- The goal is annotation or structural clustering of already-selected features rather than outcome modeling — use MamsiStructSearch instead

## Inputs

- Sample-aligned LC-MS feature-intensity CSV tables per assay block (e.g. alz_hpos.csv, alz_lpos.csv, alz_lneg.csv; or multi-assay MY Diabetes serum tables)
- Metadata CSV with a numeric phenotypic outcome column (e.g. alz_metadata.csv 'Gender' encoded 1/0, or continuous bilirubin levels)
- Train/test split specification (e.g. scikit-learn train_test_split with random_state=42)

## Outputs

- Fitted MamsiPls MB-PLS model with block scores T_, super scores Ts_, superweights W_, and block importances A_
- Estimated number of latent variables (6 LVs for the AddNeuroMed classification model)
- Held-out test-set performance metrics (accuracy 0.966, recall 1.0, specificity 0.943, F1 0.971, AUC 0.933 reported for the 6-LV AddNeuroMed model)
- Per-repeat train/test MCCV metrics (AUC, precision, recall, specificity, F1, accuracy) written to e.g. mbpls_cv_metrics.csv

## How to apply

Load each assay intensity table with pandas, adding assay prefixes (HPOS_, LPOS_, LNEG_) so feature provenance is traceable, and encode the response numerically (e.g. Female=1/Male=0); split into training and testing subsets (train_test_split, random_state=42), keeping all blocks sample-aligned by indexing on one block. Instantiate MamsiPls from the mamsi package (extends mbpls 1.0.4) with NIPALS algorithm and standardize=True, fit on the list of training blocks plus y, and estimate the number of latent variables with .estimate_lv() using 5-fold cross-validation up to max_components=10 with metric='auc' (classification) or 'q2' (regression), choosing the lowest LV at which validation performance plateaus. For regression outcomes such as bilirubin, use .evaluate_regression_model() on the held-out test set; for classification, use .evaluate_class_model(). Optionally validate robustness with .montecarlo_cv() using repeats=1000 and test_size=0.2, aggregating per-repeat train/test metrics. The rationale is that modeling interblock relationships between assay blocks improves estimation of the phenotypic outcome, and the fitted decomposition exposes block scores T_, super scores Ts_, superweights W_, and block importances A_ for downstream MB-VIP interpretation.

## Related tools

- **mamsi** (Provides MamsiPls, which extends mbpls with NIPALS MB-PLS fitting, estimate_lv, evaluate_class_model/evaluate_regression_model, montecarlo_cv, and block_importance) — https://github.com/kopeckylukas/py-mamsi
- **mbpls 1.0.4** (Pinned dependency implementing the underlying multiblock PLS algorithm) — https://github.com/kopeckylukas/py-mamsi
- **scikit-learn** (train_test_split for sample-aligned 90:10 train/test splitting of all assay blocks)
- **pandas** (Loading assay feature tables and metadata, prefixing columns, and concatenating blocks with pd.concat(axis=1))

## Examples

```
from mamsi.mamsi_pls import MamsiPls; mamsipls = MamsiPls(n_components=1); mamsipls.fit([hpos, lpos, lneg], y); mamsipls.estimate_lv([hpos, lpos, lneg], y, metric='auc')
```

## Evaluation signals

- estimate_lv cross-validation selects the lowest LV at which validation performance plateaus; for the AddNeuroMed sex-classification model this yields 6 latent variables
- Held-out test metrics should match the reported 6-LV model: accuracy 0.966, recall 1.0, specificity 0.943, F1 0.971, AUC 0.933
- MCCV with 1000 repeats (20% test per split) should give mean validation AUC consistent with the estimate_lv cross-validated performance
- All assay blocks remain sample-aligned after splitting (subsetting blocks by the index of one block); shapes of block feature matrices match the fitted model inputs
- MB-PLS performance should exceed comparable single-assay models, per the article's finding that modeling interblock relationships improved outcome estimation

## Limitations

- The MB-VIP permutation test at n_permutations=10000 is slow; the article recommends >100000 permutations, ideally run on an HPC (a permtest_hpc sample-code guide is provided)
- The PLS algorithm requires the response variable to be numeric — categorical outcomes must be encoded first (e.g. Gender as 1 for female, 0 for male)
- Framework was tested on metabolomics phenotyping data; other LC-MS data types 'should be usable' but are unvalidated
- Monte Carlo CV configuration in the tutorials is specific to Imperial HPC and may need adaptation elsewhere

## Evidence

- [abstract] We employ a multiblock-partial least-squares model (MB-PLS) coupled with multiblock variable importance in projection to estimate the importance of predictors to the outcome variable.: "We employ a multiblock-partial least-squares model (MB-PLS) coupled with multiblock variable importance in projection to estimate the importance of predictors to the outcome variable."
- [readme] Data integration analysis using the Multi-Block Partial Least Squares (MB-PLS) algorithm.: "Data integration analysis using the Multi-Block Partial Least Squares (MB-PLS) algorithm."
- [methods] mamsipls.estimate_lv([hpos_train, lpos_train, lneg_train], y_train, metric='auc'): "mamsipls.estimate_lv([hpos_train, lpos_train, lneg_train], y_train, metric='auc')"
- [methods] The LV estimation result shows that the model has 6 latent variables/components.: "The LV estimation result shows that the model has 6 latent variables/components."
- [readme] The PLS algorithm requires the response variable to be numeric.: "The PLS algorithm requires the response variable to be numeric."
