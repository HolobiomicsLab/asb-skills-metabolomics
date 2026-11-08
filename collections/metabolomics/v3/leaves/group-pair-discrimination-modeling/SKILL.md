---
name: group-pair-discrimination-modeling
description: Use when a metabolomics feature table with two or more phenotype groups
  needs binary discrimination models per group pair (e.g. CRC vs polyp vs healthy
  controls) using pre-selected metabolite features.
when_to_use_negative:
- Raw spectral data requiring peak picking and annotation — iMAP deliberately does
  not implement raw data preprocessing or peak picking/annotation modules
- Only a single phenotype group is present — binary group-pair discrimination requires
  at least two labeled groups
- The goal is unsupervised metabolite selection or correlation-based network construction
  rather than supervised classification
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: iMAP
  role: Web server providing the automatic pipeline for train-test set separation,
    feature selection, and predictive model construction and validation
  repo: https://imap.metaboprofile.cloud/
- name: randomForest
  role: R package used by iMAP's backend R scripts to fit the group-pair classifiers
    and produce OOB error rates
- name: IP4M
  role: Desktop predecessor platform of iMAP covering metabolomics data analysis from
    pre-processing to data-interpretation
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:25bcf532ef3cc2911d9dfffb937d514d4d0041cf5a5c902b8c925d590b750c4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - R
  - iMAP
  - randomForest
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Group-pair discrimination modeling

## Summary

Builds binary predictive classifiers (e.g. random forest) that discriminate between pairs of phenotype groups using selected metabolites, reporting out-of-bag (OOB) error rates per group pair. iMAP automates this pipeline from train-test separation through model construction and validation.

## When to use

Use when a metabolomics feature table with two or more phenotype groups needs binary discrimination models per group pair (e.g. CRC vs polyp vs healthy controls) using pre-selected metabolite features.

## When NOT to use

- Raw spectral data requiring peak picking and annotation — iMAP deliberately does not implement raw data preprocessing or peak picking/annotation modules
- Only a single phenotype group is present — binary group-pair discrimination requires at least two labeled groups
- The goal is unsupervised metabolite selection or correlation-based network construction rather than supervised classification

## Inputs

- Metabolomics feature table (samples x metabolites) with phenotype group labels, e.g. 234 clinical serum samples in three groups (66 CRC, 76 polyp, 92 healthy controls; Zhu et al., 2014)
- List of selected metabolite features (e.g. Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)
- Group-pair definitions (CRC-PolyP, CRC-HC, PolyP-HC)
- Train/test split specification (stratified, train:test = 8:2)

## Outputs

- Fitted random-forest classifier per group pair
- Out-of-bag error rate per group pair (e.g. 0.364 CRC-PolyP, 0.255 CRC-HC, 0.453 PolyP-HC)
- oob_error_rates.csv with one row per group pair
- Held-out test-set validation metrics contrasting pipeline variants (Pipeline-1 vs Pipeline-3)

## How to apply

Load the feature table (e.g. the Zhu et al. 2014 serum metabolomics dataset of 234 samples: 66 CRC, 76 polyp, 92 healthy controls) with a stratified train/test split (train:test = 8:2). Restrict features to metabolites selected by the metabolites-selection step (e.g. Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid) and fit a random-forest classifier (R 'randomForest' package, the backend used by iMAP's R scripts) for each group pair: CRC-PolyP, CRC-HC, and PolyP-HC. Extract the OOB error rate from each fitted randomForest object, which the bagging procedure computes automatically during training, and report one error rate per pair. Compare train-set-only OOB estimates against held-out test-set validation to detect over-fitting: the published OOB rates are 0.364 (CRC-PolyP), 0.255 (CRC-HC), and 0.453 (PolyP-HC), while pipeline comparisons show train-set-only validation over-fits relative to held-out test-set validation.

## Related tools

- **iMAP** (Web server providing the automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation) — https://imap.metaboprofile.cloud/
- **randomForest** (R package used by iMAP's backend R scripts to fit the group-pair classifiers and produce OOB error rates)
- **IP4M** (Desktop predecessor platform of iMAP covering metabolomics data analysis from pre-processing to data-interpretation) — https://github.com/IP4M

## Examples

```
library(randomForest); rf <- randomForest(y ~ Adenylosuccinate + Histidine + Hydroxyproline + LinolenicAcid, data=train_crc_hc); rf$err.rate
```

## Evaluation signals

- OOB error rates per group pair match published values: 0.364 (CRC-PolyP), 0.255 (CRC-HC), 0.453 (PolyP-HC)
- oob_error_rates.csv contains exactly one row per group pair with a numeric error rate in [0, 1]
- Train/test split is stratified and matches the 8:2 ratio used in the predictive-modeling workflow
- Held-out test-set error exceeds (i.e. is more pessimistic than) the train-set-only OOB estimate, confirming Pipeline-1 over-fits relative to Pipeline-3
- Each model uses only the four selected metabolites as features, not the full metabolite table

## Limitations

- Modules of raw data preprocessing and peak picking and annotation were not implemented in iMAP, so input must be a processed feature table
- Train-set-only OOB validation over-fits relative to held-out test-set validation, so OOB rates alone can be optimistic
- iMAP's limitations are acknowledged but not fully specified in the article's discussion

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] 234 clinical samples from three groups: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] Backend R scripts available on GitHub: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] iMAP freely available at imap.metaboprofile.cloud: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
