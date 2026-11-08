---
name: oob-error-rate-extraction
description: Use when a trained randomForest object built on metabolomics feature
  data (e.g. the four selected metabolites Adenylosuccinate, Histidine, Hydroxyproline,
  Linolenic Acid) is available and you need OOB error rates to report or compare predictive
  performance per group pair.
when_to_use_negative:
- You need unbiased generalization performance — OOB error is computed on train-set
  bagged samples and over-fits relative to iMAP's Pipeline-3 held-out test-set validation
- Your model is not a bagged random forest (e.g. PLS-DA, SVM) — no out-of-bag samples
  exist, so this error rate is undefined
- Raw spectra need preprocessing — iMAP deliberately does not implement raw data preprocessing
  or peak picking and annotation
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: randomForest
  role: R package used by iMAP's backend scripts to fit random-forest classifiers
    whose bagged-sample error rates are read as OOB error
- name: iMAP
  role: Web server whose new predictive-modeling module automates train-test separation,
    feature selection, and model construction/validation on the CRC metabolomics data
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: IP4M
  role: Desktop predecessor platform of iMAP for one-stop metabolomics data analysis,
    from which the iMAP backend R scripts derive
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:eac27ea0a0bc63b520ad8c2c3c6447899bd64f9a221946381e1dc13fba352b4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IP4M/R-scripts-for-iMAP
  tools:
  - IP4M
  - R
  - iMAP
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# oob-error-rate-extraction

## Summary

Extracts out-of-bag (OOB) error rates from random-forest classifiers trained in iMAP's predictive-modeling pipeline, enabling train-set-only validation of metabolomics group-pair discrimination (e.g. CRC vs PolyP vs HC on Zhu et al. 2014 serum metabolomics data).

## When to use

Use when a trained randomForest object built on metabolomics feature data (e.g. the four selected metabolites Adenylosuccinate, Histidine, Hydroxyproline, Linolenic Acid) is available and you need OOB error rates to report or compare predictive performance per group pair.

## When NOT to use

- You need unbiased generalization performance — OOB error is computed on train-set bagged samples and over-fits relative to iMAP's Pipeline-3 held-out test-set validation
- Your model is not a bagged random forest (e.g. PLS-DA, SVM) — no out-of-bag samples exist, so this error rate is undefined
- Raw spectra need preprocessing — iMAP deliberately does not implement raw data preprocessing or peak picking and annotation

## Inputs

- Serum metabolomics feature table from the Zhu et al. 2014 CRC dataset (234 clinical samples: 66 CRC, 76 polyp, 92 healthy controls)
- Group-pair labels (CRC-PolyP, CRC-HC, PolyP-HC) and stratified train/test split (8:2)
- Fitted randomForest R objects per group pair (trained on Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)

## Outputs

- oob_error_rates.csv with one row per group pair containing its OOB error rate (expected: 0.364 CRC-PolyP, 0.255 CRC-HC, 0.453 PolyP-HC)

## How to apply

Fit random-forest classifiers with the R 'randomForest' package (the backend used by iMAP's R scripts) on each group pair from the 234-sample colorectal cancer dataset (66 CRC, 76 polyp, 92 healthy controls; Zhu et al. 2014), using an 8:2 stratified train:test split as in iMAP's predictive-modeling workflow. During training, the randomForest algorithm automatically evaluates each tree on the bagged (out-of-bag) samples, so the OOB error rate is available directly on the fitted object without a separate prediction step. Extract the OOB error per pair (CRC-PolyP, CRC-HC, PolyP-HC) and write one row per pair (e.g. to oob_error_rates.csv). Validate against published values of 0.364 (CRC-PolyP), 0.255 (CRC-HC), and 0.453 (PolyP-HC); note that these train-set-only OOB rates over-fit relative to held-out test-set (Pipeline-3) validation.

## Related tools

- **randomForest** (R package used by iMAP's backend scripts to fit random-forest classifiers whose bagged-sample error rates are read as OOB error)
- **iMAP** (Web server whose new predictive-modeling module automates train-test separation, feature selection, and model construction/validation on the CRC metabolomics data) — https://github.com/IP4M/R-scripts-for-iMAP
- **IP4M** (Desktop predecessor platform of iMAP for one-stop metabolomics data analysis, from which the iMAP backend R scripts derive) — https://github.com/IP4M

## Examples

```
library(randomForest); rf <- randomForest(group ~ Adenylosuccinate + Histidine + Hydroxyproline + LinolenicAcid, data=train, ntree=500); oob <- rf$err.rate[rf$ntree, "OOB"]
```

## Evaluation signals

- The extracted OOB error rates must equal the published values: 0.364 (CRC-PolyP), 0.255 (CRC-HC), 0.453 (PolyP-HC)
- oob_error_rates.csv contains exactly three rows, one per group pair (CRC-PolyP, CRC-HC, PolyP-HC)
- All error rates fall in the plausible range [0, 1] and are consistent with the 8:2 stratified train:test split of 234 samples
- Each randomForest object was trained only on the four selected metabolites (Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)

## Limitations

- OOB (train-set-only) validation over-fits relative to held-out test-set validation, as shown by the Pipeline-1 vs Pipeline-3 comparison
- iMAP does not implement raw data preprocessing or peak picking and annotation modules, so inputs must already be quantified feature tables
- Backend behavior depends on the R-scripts-for-iMAP repository; no changelog was found for it

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] Dataset description: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] Backend R scripts: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] README availability statement: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
