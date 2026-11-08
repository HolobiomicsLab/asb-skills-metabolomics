---
name: serum-metabolite-biomarker-classification
description: 'Use when a labeled metabolomics feature table (e.g., 234 serum samples:
  66 CRC, 76 polyp, 92 healthy controls from Zhu et al., 2014) must be turned into
  a validated group-pair classifier with OOB and held-out test error rates.'
when_to_use_negative:
- 'Raw spectrometry data before preprocessing: iMAP does not implement raw data preprocessing
  or peak picking/annotation modules, so a processed feature table must be provided
  first'
- Unsupervised metabolomics analysis goals (clustering, enrichment, pathway interpretation)
  where no phenotype classifier is required
- Datasets too small to support a stratified train/test split and reliable OOB estimation
edam_operation: http://edamontology.org/operation_3948
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server whose new module automates train-test set separation, feature selection,
    and predictive model construction and validation
  repo: https://github.com/IP4M
- name: IP4M
  role: Desktop predecessor platform covering metabolomics analysis steps from pre-processing
    to data-interpretation
  repo: https://github.com/IP4M
- name: randomForest
  role: R package used by iMAP's backend R scripts to fit random-forest classifiers
    and provide OOB error rates
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: R-scripts-for-iMAP
  role: Backend R scripts implementing the predictive-modeling pipeline
  repo: https://github.com/IP4M/R-scripts-for-iMAP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:19b6f445343a895f4939238449dac56347ad37a10600db7ab82da64544bef758
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - R
  - R-scripts-for-iMAP
  - iMAP
  - randomForest
  edam_operation: http://edamontology.org/operation_3948
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# serum-metabolite-biomarker-classification

## Summary

Build and validate predictive metabolomics classifiers (e.g., random-forest models on selected serum metabolites for CRC vs polyp vs healthy groups) using iMAP's dedicated module, which automates train-test separation, feature selection, and model construction/validation.

## When to use

Use when a labeled metabolomics feature table (e.g., 234 serum samples: 66 CRC, 76 polyp, 92 healthy controls from Zhu et al., 2014) must be turned into a validated group-pair classifier with OOB and held-out test error rates.

## When NOT to use

- Raw spectrometry data before preprocessing: iMAP does not implement raw data preprocessing or peak picking/annotation modules, so a processed feature table must be provided first
- Unsupervised metabolomics analysis goals (clustering, enrichment, pathway interpretation) where no phenotype classifier is required
- Datasets too small to support a stratified train/test split and reliable OOB estimation

## Inputs

- Metabolomics feature table (samples x metabolites) with phenotype labels, e.g., 234 serum samples (66 CRC, 76 polyp, 92 healthy controls; Zhu et al., 2014)
- List of selected discriminatory metabolites (e.g., Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)
- Train/test split configuration (e.g., stratified 8:2 train:test)

## Outputs

- Fitted random-forest classifiers per group pair (CRC-PolyP, CRC-HC, PolyP-HC)
- Out-of-bag (OOB) error rate per group pair, e.g., written to oob_error_rates.csv
- Held-out test-set validation error rates for comparison against train-set-only OOB rates

## How to apply

Load the metabolomics feature table with phenotype labels and apply iMAP's predictive-modeling module, which runs an automatic pipeline of train-test set separation (e.g., stratified 8:2 split), feature selection, and model construction and validation. For each group pair (CRC-PolyP, CRC-HC, PolyP-HC), fit a random-forest classifier (R 'randomForest' package, the backend of iMAP's R scripts) on the selected metabolites such as Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid. Extract the out-of-bag (OOB) error rate from each randomForest object for train-set-only validation, and evaluate the held-out test set to quantify over-fitting (Pipeline-1 vs Pipeline-3 comparison). Write per-pair error rates (expected OOB: 0.364 CRC-PolyP, 0.255 CRC-HC, 0.453 PolyP-HC) to a CSV for reproducibility checks.

## Related tools

- **iMAP** (Web server whose new module automates train-test set separation, feature selection, and predictive model construction and validation) — https://github.com/IP4M
- **IP4M** (Desktop predecessor platform covering metabolomics analysis steps from pre-processing to data-interpretation) — https://github.com/IP4M
- **randomForest** (R package used by iMAP's backend R scripts to fit random-forest classifiers and provide OOB error rates) — https://github.com/IP4M/R-scripts-for-iMAP
- **R-scripts-for-iMAP** (Backend R scripts implementing the predictive-modeling pipeline) — https://github.com/IP4M/R-scripts-for-iMAP

## Examples

```
Rscript -e 'library(randomForest); fit <- randomForest(y ~ ., data=train, ntree=500); fit$err.rate["OOB"]'
```

## Evaluation signals

- OOB error rates match published values: 0.364 (CRC-PolyP), 0.255 (CRC-HC), 0.453 (PolyP-HC)
- oob_error_rates.csv contains one row per group pair (CRC-PolyP, CRC-HC, PolyP-HC) with a numeric error rate
- Train-set OOB error is markedly lower than held-out test-set error, confirming Pipeline-1 (train-set-only validation) over-fits relative to Pipeline-3
- Train/test split is stratified with the expected 8:2 ratio across the 66 CRC / 76 polyp / 92 healthy-control labels
- Model features are restricted to the selected metabolites (Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)

## Limitations

- iMAP does not implement raw data preprocessing or peak picking and annotation modules, considering the risk of data security; input must already be a processed feature table
- OOB error alone (train-set-only validation, Pipeline-1) over-fits relative to held-out test-set validation (Pipeline-3), as shown by the CRC group-pair comparison
- iMAP has acknowledged but unspecified limitations per the discussion section

## Evidence

- [abstract] new module with automatic pipeline: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [intro] raw preprocessing not implemented: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [methods] dataset composition: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] backend R scripts: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] iMAP availability: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
