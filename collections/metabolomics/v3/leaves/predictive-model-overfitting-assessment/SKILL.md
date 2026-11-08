---
name: predictive-model-overfitting-assessment
description: Use when building a metabolomics classifier (e.g. random forest on feature-selected
  metabolites) and you need to measure how much train-set-only validation (OOB error)
  underestimates generalization error versus a stratified held-out test set.
when_to_use_negative:
- Raw spectral data (mzML/NetCDF) — iMAP does not implement raw data preprocessing
  or peak picking and annotation modules, so feature extraction must be done elsewhere
  first
- Feature table has not passed iMAP's data-processing filter steps (e.g. removing
  samples with >90% zeros and variables with >40% zeros, normalization to counts per
  million), which would bias the classifier
- The analysis goal is differential-abundance testing (metabolites-selection) rather
  than predictive model construction and validation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: iMAP
  role: Web server providing the automatic pipeline for train-test set separation,
    feature selection, and predictive model construction and validation
  repo: https://imap.metaboprofile.cloud/
- name: randomForest
  role: R package used by iMAP's backend R scripts to fit random-forest classifiers
    whose OOB error rates quantify train-set over-fitting
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: IP4M
  role: Desktop predecessor platform of iMAP covering the metabolomics analysis workflow
    from pre-processing to data-interpretation
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:6e313eb62b7f350161d28be2e8f5e1184702c2eaf402ec2015e10a88dfaba00d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - R
  - iMAP
  - randomForest
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# predictive-model-overfitting-assessment

## Summary

Quantifies over-fitting in metabolomics predictive models by comparing train-set-only validation (e.g. random-forest OOB error on selected metabolites) against held-out test-set validation, using iMAP's automatic train-test separation pipeline on the Zhu et al. (2014) CRC serum metabolomics dataset.

## When to use

Use when building a metabolomics classifier (e.g. random forest on feature-selected metabolites) and you need to measure how much train-set-only validation (OOB error) underestimates generalization error versus a stratified held-out test set.

## When NOT to use

- Raw spectral data (mzML/NetCDF) — iMAP does not implement raw data preprocessing or peak picking and annotation modules, so feature extraction must be done elsewhere first
- Feature table has not passed iMAP's data-processing filter steps (e.g. removing samples with >90% zeros and variables with >40% zeros, normalization to counts per million), which would bias the classifier
- The analysis goal is differential-abundance testing (metabolites-selection) rather than predictive model construction and validation

## Inputs

- Serum metabolomics feature table for 234 clinical samples (66 CRC, 76 polyp, 92 healthy controls; Zhu et al., 2014)
- Group-pair class labels (CRC vs PolyP, CRC vs HC, PolyP vs HC)
- Feature-selected metabolites: Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid

## Outputs

- oob_error_rates.csv with one row per group pair (CRC-PolyP, CRC-HC, PolyP-HC)
- Fitted randomForest objects with OOB error rates per group pair

## How to apply

Load the serum metabolomics data of 234 clinical samples (66 CRC, 76 polyp, 92 healthy controls; Zhu et al., 2014) and apply iMAP's automatic pipeline for train-test set separation using a stratified 8:2 train:test split. Fit random-forest classifiers with the R 'randomForest' package (as used by iMAP's backend R scripts at github.com/IP4M/R-scripts-for-iMAP) on the four selected metabolites (Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid) for each group pair: CRC-PolyP, CRC-HC, and PolyP-HC. Extract the out-of-bag (OOB) error rate from each fitted randomForest object, which is computed automatically from the out-of-bag bagged samples during training. Write one OOB error rate per group pair to oob_error_rates.csv. The rationale is that Pipeline-1 (train-set-only OOB validation) over-fits relative to Pipeline-3 (held-out test-set validation), so the OOB rates serve as the optimistic lower bound on error.

## Related tools

- **iMAP** (Web server providing the automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation) — https://imap.metaboprofile.cloud/
- **randomForest** (R package used by iMAP's backend R scripts to fit random-forest classifiers whose OOB error rates quantify train-set over-fitting) — https://github.com/IP4M/R-scripts-for-iMAP
- **IP4M** (Desktop predecessor platform of iMAP covering the metabolomics analysis workflow from pre-processing to data-interpretation) — https://github.com/IP4M

## Examples

```
Rscript -e 'library(randomForest); fit <- randomForest(y ~ ., data=train_set, subset=select=c("Adenylosuccinate","Histidine","Hydroxyproline","Linolenic_Acid")); write.csv(fit$err.rate, "oob_error_rates.csv")'
```

## Evaluation signals

- OOB error rates must match the published values 0.364 (CRC-PolyP), 0.255 (CRC-HC), and 0.453 (PolyP-HC) from the paper
- OOB error rates (Pipeline-1, train-set-only) should be lower than held-out test-set error rates (Pipeline-3), confirming over-fitting of train-set validation
- oob_error_rates.csv contains exactly three rows, one per group pair (CRC-PolyP, CRC-HC, PolyP-HC)
- Train/test split is stratified 8:2 and applied before feature selection, matching iMAP's automatic pipeline
- All three random-forest fits use only the four selected metabolites, not the full feature table

## Limitations

- Train-set-only validation (OOB error) over-fits relative to held-out test-set validation, so OOB rates alone understate generalization error
- Modules of raw data preprocessing and peak picking and annotation were not implemented in iMAP, considering the risk of data security
- Version numbers for iMAP/IP4M are not stated and no changelog is available for the R-scripts-for-iMAP repository, complicating reproducibility pinning

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] backend R scripts location: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [methods] dataset description: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [intro] unimplemented modules: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
