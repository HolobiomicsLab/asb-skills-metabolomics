---
name: statistical-hypothesis-testing-on-importance-distributions
description: Use when you have trained per-target random forest models (e.g., one
  per metabolite via 5-fold CV) and need to statistically test whether a pre-selected
  subset of top features is more important than the full feature set in independent
  validation data.
when_to_use_negative:
- Do not use to claim the CCLE-trained models transfer directly to other data sources
  — the article notes such models are not directly transferrable due to differences
  between in vitro and in vivo conditions, cell line panels, and limited feature overlap.
- Do not interpret a significant t-test/KS result as a causal or mechanistic relationship
  — the 'guilt by association' strategy may not identify causative mechanisms.
- Do not apply to a single metabolite's feature importances without pooling; the tests
  compare pooled importance distributions, not individual models.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: machine learning (random forest regression)
  role: Random forest regressors are trained per metabolite in the NCI60 and single-cell
    validation datasets to produce the feature importances being tested
- name: scikit-learn
  role: RandomForestRegressor is used to train the per-metabolite models and extract
    feature importances from each
- name: Python
  role: Implementation language for the RF training, feature-importance extraction,
    and t-test/KS test workflow
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:95af9e5a711361a24bf95e154ab035015b27bad60eabec5f225e2a3747bd057d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# statistical-hypothesis-testing-on-importance-distributions

## Summary

A generalizability validation skill from Recon8D that tests whether CCLE-identified top metabolite predictors show significantly higher random-forest feature importances in independent NCI60 and single-cell datasets than the full feature set, using t-tests and KS tests on pooled importance distributions.

## When to use

Use when you have trained per-target random forest models (e.g., one per metabolite via 5-fold CV) and need to statistically test whether a pre-selected subset of top features is more important than the full feature set in independent validation data.

## When NOT to use

- Do not use to claim the CCLE-trained models transfer directly to other data sources — the article notes such models are not directly transferrable due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap.
- Do not interpret a significant t-test/KS result as a causal or mechanistic relationship — the 'guilt by association' strategy may not identify causative mechanisms.
- Do not apply to a single metabolite's feature importances without pooling; the tests compare pooled importance distributions, not individual models.

## Inputs

- NCI60 omics data downloaded from the NCI CellMiner portal
- single-cell cell-cycle transcriptomics data
- list of CCLE-identified top features isolated from the CCLE-trained metabolite models (e.g., Metab8D_network.xlsx top-20 features)
- full feature set for each validation dataset (NaNs zero-imputed, zero-variance features removed)

## Outputs

- pooled feature-importance distributions for the top-feature subset and the full feature set per dataset
- t-test statistics and p-values comparing the two importance distributions
- Kolmogorov-Smirnov (KS) test statistics and p-values
- validation tables replicating Supplementary Tables 3-5

## How to apply

First preprocess the validation data (NCI60 from the NCI CellMiner portal and single-cell cell-cycle transcriptomics): replace missing NaN values with zeros and remove zero-variance features. Train a separate scikit-learn RandomForestRegressor for each metabolite using 5-fold cross-validation with Z-score normalization computed within each CV fold, mirroring the CCLE-trained model setup. Extract feature importances (random forest importance scores or Shapley values) from each trained model, then pool the importance scores of the CCLE-identified top features versus those of the full feature set within each dataset. Apply both a t-test and a Kolmogorov-Smirnov test to the two pooled importance distributions to assess whether top features are significantly more important than expected by chance. Report test statistics and p-values in tables replicating Supplementary Tables 3-5.

## Related tools

- **machine learning (random forest regression)** (Random forest regressors are trained per metabolite in the NCI60 and single-cell validation datasets to produce the feature importances being tested)
- **scikit-learn** (RandomForestRegressor is used to train the per-metabolite models and extract feature importances from each)
- **Python** (Implementation language for the RF training, feature-importance extraction, and t-test/KS test workflow) — https://github.com/sriram-lab/Metab8D

## Examples

```
from scipy.stats import ttest_ind, ks_2samp; t, p_t = ttest_ind(top_imp, full_imp); d, p_ks = ks_2samp(top_imp, full_imp)
```

## Evaluation signals

- Both t-test and KS test on the pooled importance distributions yield significant p-values, indicating CCLE top features exceed the full feature set in importance.
- Feature importances are extracted from a separately trained RandomForestRegressor for every metabolite, not from a single pooled model.
- Data preprocessing matches the article: missing NaNs replaced with zeros and zero-variance features removed before model training.
- Z-score normalization is computed within each CV fold, not on the full dataset prior to splitting.
- Reported statistics and p-values are formatted to replicate Supplementary Tables 3-5.

## Limitations

- CCLE-trained models are not directly transferrable to other metabolomics sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap.
- Network associations driven by systems-level data are descriptive, and findings are not necessarily mechanistic ('guilt by association').
- Confidence scores in the network output depend on 8 control experiments, and the top-20 features per metabolite come from nine feature sets (genomic mutations excluded from RF results due to their binary structure).

## Evidence
