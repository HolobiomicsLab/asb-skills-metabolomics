---
name: stratified-train-test-split-metabolomics
description: Use when building predictive models (e.g. RF or elastic-net classifiers)
  from a metabolite-by-sample intensity table with class labels, and you need class-proportioned
  train/test separation before feature selection and AUC evaluation.
when_to_use_negative:
- Raw spectra need preprocessing or peak picking/annotation — iMAP deliberately does
  not implement raw data preprocessing or peak picking and annotation modules
- The task is pure statistical metabolite selection or network analysis without any
  prediction/validation goal — no train-test separation is needed
- The dataset is too small for a 20% hold-out to preserve class representation; nested
  cross-validation may be more appropriate
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: iMAP
  role: Web server whose newly developed predictive-model module performs the automatic
    train-test set separation step
  repo: https://imap.metaboprofile.cloud/
- name: IP4M/R-scripts-for-iMAP
  role: Backend R scripts implementing the train-test separation and downstream modeling
    steps
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: pROC
  role: R package used to compute ROC AUCs on both train and test sets after the split
- name: random Forest
  role: Classifier fitted on the train set after the stratified split, for two-group
    pair comparisons
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:0c83e6bb929e7049e2ec5df5c72dbc3e2874fd08ffeeda2c2cd967d30d67b98e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M/R-scripts-for-iMAP
  - iMAP
  - pROC
  - random Forest
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Stratified train-test split for metabolomics predictive modeling

## Summary

Splits a metabolomics intensity matrix into stratified training and held-out test sets (e.g. 8:2 ratio on the 234-sample Zhu et al. 2014 serum metabolite dataset: 66 CRC, 76 PolyP, 92 HC) as the first step of iMAP's automatic predictive-modeling pipeline, so that feature selection and model fitting can be followed by honest validation on unseen samples.

## When to use

Use when building predictive models (e.g. RF or elastic-net classifiers) from a metabolite-by-sample intensity table with class labels, and you need class-proportioned train/test separation before feature selection and AUC evaluation.

## When NOT to use

- Raw spectra need preprocessing or peak picking/annotation — iMAP deliberately does not implement raw data preprocessing or peak picking and annotation modules
- The task is pure statistical metabolite selection or network analysis without any prediction/validation goal — no train-test separation is needed
- The dataset is too small for a 20% hold-out to preserve class representation; nested cross-validation may be more appropriate

## Inputs

- Targeted serum metabolite intensity matrix (samples x metabolites; 234 samples, 113 metabolites)
- Class/group labels per sample (e.g. 66 CRC, 76 PolyP, 92 HC)
- Train-test split ratio (8:2) and stratification variable (group)

## Outputs

- Stratified training set (80% of samples per group) for feature selection and model fitting
- Held-out stratified test set (20% of samples per group) for validation AUC computation
- AUC table (auc_table.csv) with group pair, method (random forest / elastic net), AUC in test set, AUC in train set

## How to apply

Load the targeted serum metabolite intensity matrix (234 samples: 66 colorectal cancer, 76 polyp, 92 healthy controls; 113 metabolites; Zhu et al., 2014). Split samples into train and test sets at an 8:2 ratio using stratified random sampling so each group (CRC, PolyP, HC) keeps its class proportions. Perform all subsequent steps of iMAP's pipeline — metabolite selection (pairwise Tukey post-hoc P < 0.05 after Kruskal-Wallis and |log2FC| > 0.25) and fitting of random forest and elastic-net classifiers — using only the train set. Reserve the held-out 20% test set exclusively for validation ROC AUC computation with pROC, so train-set AUC and test-set AUC can be compared (train AUC is expected to exceed test AUC, as in the paper's Table 2, e.g. RF HC vs. CRC 1 train / 0.743 test).

## Related tools

- **iMAP** (Web server whose newly developed predictive-model module performs the automatic train-test set separation step) — https://imap.metaboprofile.cloud/
- **IP4M/R-scripts-for-iMAP** (Backend R scripts implementing the train-test separation and downstream modeling steps) — https://github.com/IP4M/R-scripts-for-iMAP
- **pROC** (R package used to compute ROC AUCs on both train and test sets after the split)
- **random Forest** (Classifier fitted on the train set after the stratified split, for two-group pair comparisons)

## Examples

```
Rscript imap_train_test_split.R --input zhu2014_serum_113metabolites.csv --groups CRC,PolyP,HC --ratio 8:2 --stratify group --seed 42
```

## Evaluation signals

- Stratification check: group proportions (66 CRC, 76 PolyP, 92 HC) preserved at ~8:2 ratio in train and test sets
- Compiled auc_table.csv reproduces the reported Table 2 values (e.g. RF HC vs. CRC 0.743 test / 1 train; elastic net HC vs. PolyP 0.5 test / 0.5 train)
- AUC in train set exceeds AUC in test set in all cases except elastic net between HC and PolyP
- Feature selection and model fitting consume only the train split; the test split is touched only at validation time
- Exactly four metabolites (Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid) pass the Tukey P < 0.05 and |log2FC| > 0.25 selection criteria on the train set

## Limitations

- Demonstrated on a single targeted serum metabolomics dataset (Zhu et al., 2014) with three groups; behavior on unbalanced or multi-class problems is not characterized
- iMAP does not implement raw data preprocessing or peak picking/annotation, so the intensity matrix must already be preprocessed upstream
- The paper acknowledges 'there are still many limitations' with iMAP without specifying them; no changelog exists for the associated repository
- No version number is stated for iMAP or IP4M

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] 234 clinical samples from three groups: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] Backend R scripts are available: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] iMAP described in README as web server with predictive model module: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [abstract] iMAP availability: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
