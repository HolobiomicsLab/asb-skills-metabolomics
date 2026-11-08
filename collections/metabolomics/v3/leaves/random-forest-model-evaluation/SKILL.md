---
name: random-forest-model-evaluation
description: Use when a metabolomics feature table with phenotype labels (e.g. CRC,
  polyp, healthy controls) needs supervised classification and validation of predictive
  performance via random forest OOB error rates.
when_to_use_negative:
- Raw spectral data requiring preprocessing — iMAP explicitly did not implement 'raw
  data preprocessing and peak picking and annotation' modules, so peak-picked feature
  tables are required
- Analyses where the goal is unsupervised metabolite selection rather than supervised
  classification with known group labels
- Datasets where the aim is network construction or differential-variable selection
  rather than predictive model construction and validation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: iMAP
  role: Web server providing the automatic predictive-modeling pipeline (train-test
    set separation, feature selection, predictive model construction and validation)
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Predecessor desktop platform; iMAP is the updated web-server version of IP4M
  repo: https://github.com/IP4M
- name: randomForest
  role: R package used by iMAP's backend R scripts to fit the random-forest classifiers
    whose OOB error rates are evaluated
  repo: https://github.com/IP4M/R-scripts-for-iMAP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:ccbd7dd9328f798b10a2e4fdca915aa1696f708d29f273c1c7d2cab7d4c8289b
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# random-forest-model-evaluation

## Summary

Reproduces random-forest classifier training and out-of-bag (OOB) error-rate evaluation for metabolomics group-pair classification, as implemented by iMAP's predictive-modeling module on the Zhu et al. (2014) CRC serum metabolomics dataset.

## When to use

Use when a metabolomics feature table with phenotype labels (e.g. CRC, polyp, healthy controls) needs supervised classification and validation of predictive performance via random forest OOB error rates.

## When NOT to use

- Raw spectral data requiring preprocessing — iMAP explicitly did not implement 'raw data preprocessing and peak picking and annotation' modules, so peak-picked feature tables are required
- Analyses where the goal is unsupervised metabolite selection rather than supervised classification with known group labels
- Datasets where the aim is network construction or differential-variable selection rather than predictive model construction and validation

## Inputs

- Serum metabolomics feature table of 234 clinical samples (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls; Zhu et al., 2014)
- Selected metabolite features (e.g. Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)
- Group-pair phenotype labels (CRC-PolyP, CRC-HC, PolyP-HC)
- Train/test stratified split (train:test = 8:2)

## Outputs

- Fitted randomForest classifier objects per group pair
- Out-of-bag (OOB) error rate per group pair
- CSV file of OOB error rates with one row per group pair (e.g. oob_error_rates.csv)

## How to apply

Load the serum metabolomics data of the 234 clinical samples (66 CRC, 76 polyp, 92 healthy controls; Zhu et al., 2014) and apply the stratified train/test split (train:test = 8:2) used in iMAP's predictive-modeling workflow. Fit random-forest classifiers with the R 'randomForest' package (the backend of iMAP's R scripts) on the four selected metabolites (Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid) for each group pair: CRC-PolyP, CRC-HC, and PolyP-HC. Extract the out-of-bag (OOB) error rate from each fitted randomForest object; the OOB rate is computed automatically from the bagged (out-of-bag) samples during training, so no separate OOB computation is needed. Compare train-set-only validation (Pipeline-1) against held-out test-set validation (Pipeline-3) to detect over-fitting. Correct application is judged when the OOB rates match the published values 0.364 (CRC-PolyP), 0.255 (CRC-HC), and 0.453 (PolyP-HC).

## Related tools

- **iMAP** (Web server providing the automatic predictive-modeling pipeline (train-test set separation, feature selection, predictive model construction and validation)) — https://imap.metaboprofile.cloud/
- **IP4M** (Predecessor desktop platform; iMAP is the updated web-server version of IP4M) — https://github.com/IP4M
- **randomForest** (R package used by iMAP's backend R scripts to fit the random-forest classifiers whose OOB error rates are evaluated) — https://github.com/IP4M/R-scripts-for-iMAP

## Examples

```
Rscript -e 'library(randomForest); fit <- randomForest(y ~ Adenylosuccinate + Histidine + Hydroxyproline + Linolenic_Acid, data=train_set); fit$err.rate[fit$ntree, "OOB"]'
```

## Evaluation signals

- OOB error rate for CRC-PolyP equals 0.364
- OOB error rate for CRC-HC equals 0.255
- OOB error rate for PolyP-HC equals 0.453
- oob_error_rates.csv contains exactly one row per group pair (CRC-PolyP, CRC-HC, PolyP-HC)
- Train-set-only validation (Pipeline-1) error rates are demonstrably lower than held-out test-set validation (Pipeline-3), confirming over-fitting of train-set-only validation

## Limitations

- Pipeline-1 (train-set-only validation) over-fits relative to Pipeline-3 (held-out test-set validation); OOB error alone on the training set is not sufficient performance evidence
- Backend R scripts are provided as-is at github.com/IP4M/R-scripts-for-iMAP with no changelog found in the associated repository
- Raw data preprocessing, peak picking, and annotation were not implemented in iMAP considering the risk of data security

## Evidence

- [abstract] A new module provides an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] 234 clinical samples CRC dataset: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] Backend R scripts location: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [intro] Raw preprocessing not implemented: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [abstract] iMAP web server availability: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
