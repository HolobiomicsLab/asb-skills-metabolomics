---
name: random-forest-classifier-training
description: Use when a metabolomics intensity matrix (e.g., 234 samples, 113 metabolites,
  3 phenotype groups) must be split into train/test sets, modeled with random forest,
  and validated via ROC AUC against a held-out test set.
when_to_use_negative:
- When raw spectra must first be peak-picked and annotated — iMAP does not implement
  raw data preprocessing or peak picking and annotation modules
- When the goal is unsupervised metabolite selection without a phenotype label; feature
  selection here requires group pairs and Kruskal-Wallis/Tukey comparisons
- When only a single train-set AUC (the paper's own pipeline) suffices and no held-out
  validation is needed
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server whose new module provides the automatic pipeline for train-test
    set separation, feature selection, and predictive model construction and validation
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Desktop predecessor platform hosting the backend R scripts for the iMAP modeling
    pipeline
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: random Forest
  role: Classifier fitted on the selected metabolites (e.g., 4 metabolites per group
    pair) in the train set
- name: pROC
  role: R package used to compute ROC AUCs for both train-set and held-out test-set
    validation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:b3328da078bebf1b75e4a53adc0e326c4406dc003cbc43336131392391d9258c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - iMAP
  - pROC
  - random Forest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# random-forest-classifier-training

## Summary

Train and validate random forest classifiers on selected metabolite features using iMAP's predictive-modeling module, which automates train-test separation, feature selection, model construction, and ROC AUC validation (demonstrated on the Zhu et al. 2014 colorectal cancer metabolite dataset).

## When to use

Use when a metabolomics intensity matrix (e.g., 234 samples, 113 metabolites, 3 phenotype groups) must be split into train/test sets, modeled with random forest, and validated via ROC AUC against a held-out test set.

## When NOT to use

- When raw spectra must first be peak-picked and annotated — iMAP does not implement raw data preprocessing or peak picking and annotation modules
- When the goal is unsupervised metabolite selection without a phenotype label; feature selection here requires group pairs and Kruskal-Wallis/Tukey comparisons
- When only a single train-set AUC (the paper's own pipeline) suffices and no held-out validation is needed

## Inputs

- Targeted serum metabolite intensity matrix (samples x metabolites, e.g., 234 samples x 113 metabolites from Zhu et al., 2014)
- Phenotype group labels per sample (CRC, PolyP, HC)
- Train/test split specification (8:2 ratio, stratified random sampling)

## Outputs

- Fitted random forest classifier per group pair
- auc_table.csv recording group pair, method, AUC in test set, and AUC in train set (six AUC values per method)
- ROC AUC values computed with pROC for train and test pipelines

## How to apply

Load the targeted serum metabolite intensity matrix (234 samples: 66 CRC, 76 PolyP, 92 HC; 113 metabolites) and split into train and test sets at an 8:2 ratio using stratified random sampling. Select features from the train set only, requiring pairwise Tukey post-hoc P < 0.05 in at least one group pair after Kruskal-Wallis and |log2FC| > 0.25 for both CRC vs. PolyP and CRC vs. HC (yielding Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid). Fit the random forest classifier on the train set for each group pair (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP). Validate twice with pROC-computed ROC AUCs: once on the train set (the paper's pipeline) and once on the held-out test set, then compile group pair, method, test AUC, and train AUC into auc_table.csv.

## Related tools

- **iMAP** (Web server whose new module provides the automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation) — https://imap.metaboprofile.cloud/
- **IP4M** (Desktop predecessor platform hosting the backend R scripts for the iMAP modeling pipeline) — https://github.com/IP4M/R-scripts-for-iMAP
- **random Forest** (Classifier fitted on the selected metabolites (e.g., 4 metabolites per group pair) in the train set)
- **pROC** (R package used to compute ROC AUCs for both train-set and held-out test-set validation)

## Examples

```
rf <- randomForest::randomForest(y ~ ., data = train[, c("Adenylosuccinate","Histidine","Hydroxyproline","Linolenic_Acid","y")]); auc <- pROC::roc(test$y, predict(rf, test, type = "prob")[,2])$auc
```

## Evaluation signals

- auc_table.csv must match the reported Table 2 values, e.g., RF HC vs. CRC: 0.743 test / 1 train
- AUC in the train set should exceed AUC in the test set in all cases except elastic net between HC and PolyP (where both equal 0.5)
- Feature selection must yield exactly four metabolites: Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid, using train-set data only
- Train/test split must be 8:2 with stratified random sampling preserving the 66 CRC / 76 PolyP / 92 HC composition
- Twelve AUC values total in the table: six per method (three group pairs x two validation pipelines)

## Limitations

- Raw data preprocessing and peak picking and annotation modules were not implemented in iMAP, so inputs must already be intensity matrices
- Overfitting is detectable as a large train-test AUC gap (e.g., RF train AUC = 1 vs. test AUC = 0.743), so test-set AUC should be treated as the honest estimate
- Original contributions are available only in the article/Supplementary Material with inquiries via corresponding authors rather than a public data deposit

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [supplementary] split samples into train and test sets at an 8:2 ratio using stratified random sampling: [evidence span withheld — anchor status: not_found]
- [supplementary] pairwise Tukey post-hoc P < 0.05 in at least one group pair after Kruskal-Wallis test, and |log2FC| > 0.25: [evidence span withheld — anchor status: not_found]
- [supplementary] the table must match the reported Table 2 values (e.g., RF HC vs. CRC 0.743 test / 1 train: [evidence span withheld — anchor status: not_found]
- [methods] The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
