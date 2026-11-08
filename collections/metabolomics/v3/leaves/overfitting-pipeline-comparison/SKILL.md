---
name: overfitting-pipeline-comparison
description: Use when a metabolomics predictive model (e.g., RF or elastic net on
  selected metabolites) must be validated with a train-vs-test AUC comparison to quantify
  overfitting, as in iMAP's new model-building module.
when_to_use_negative:
- When the goal is raw-data preprocessing (missing values, normalization, batch correction)
  — iMAP explicitly did not implement raw-data preprocessing or peak picking/annotation
  modules
- When only a single validation pipeline is required and no train-vs-test overfitting
  comparison is intended; a plain cross-validated AUC suffices
- When the dataset lacks an independent held-out test set, making the train-vs-test
  contrast impossible
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: iMAP
  role: Web server providing the automatic pipeline for train-test set separation,
    feature selection, and predictive model construction and validation
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Predecessor desktop platform whose R backend underlies iMAP's model-building
    workflow
  repo: https://github.com/IP4M
- name: random Forest
  role: Tree-ensemble classifier fitted on the selected metabolites for each group
    pair
- name: pROC
  role: R package used to compute ROC AUCs in both the train-set and test-set validation
    pipelines
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:8bb1e236265fef0d05eb6ea288c26e7320ddb40024bcd10eea345f842681c46c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - iMAP
  - pROC
  - random Forest
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# overfitting-pipeline-comparison

## Summary

A validation skill from iMAP's predictive-modeling module: fit classifiers (random forest and elastic net) once, then compute ROC AUC in both the train set and a held-out test set, and compare the two pipelines to expose overfitting (train AUC ≫ test AUC).

## When to use

Use when a metabolomics predictive model (e.g., RF or elastic net on selected metabolites) must be validated with a train-vs-test AUC comparison to quantify overfitting, as in iMAP's new model-building module.

## When NOT to use

- When the goal is raw-data preprocessing (missing values, normalization, batch correction) — iMAP explicitly did not implement raw-data preprocessing or peak picking/annotation modules
- When only a single validation pipeline is required and no train-vs-test overfitting comparison is intended; a plain cross-validated AUC suffices
- When the dataset lacks an independent held-out test set, making the train-vs-test contrast impossible

## Inputs

- Targeted serum metabolite intensity matrix (samples x metabolites, e.g., 234 samples x 113 metabolites with group labels CRC/PolyP/HC)
- Group-pair definitions for binary classification (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP)
- Train/test split specification (8:2, stratified random sampling)
- Feature-selection criteria (Tukey post-hoc P < 0.05 threshold; |log2FC| > 0.25)

## Outputs

- auc_table.csv with columns: group pair, method, AUC in test set, AUC in train set (six AUC values per method)
- ROC curves computed with pROC for each model in each validation pipeline
- Overfitting diagnosis per model (train AUC vs. test AUC comparison)

## How to apply

Split the metabolite intensity matrix into train and test sets at an 8:2 ratio using stratified random sampling (e.g., 234 samples: 66 CRC, 76 PolyP, 92 HC; 113 metabolites from Zhu et al., 2014). Select features from the train set only, using pairwise Tukey post-hoc P < 0.05 in at least one group pair after Kruskal-Wallis plus |log2FC| > 0.25 for CRC vs. PolyP and CRC vs. HC (this yields Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid). Fit a random forest and an elastic-net (L1+L2) classifier per group pair (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP), then validate each model twice with pROC: once on the train set (the paper's pipeline) and once on the held-out test set. Compile AUCs into a table with group pair, method, test-set AUC, and train-set AUC. Rationale: train AUC exceeding test AUC signals overfitting; the paper's expectation is that train AUC > test AUC in all cases except elastic net between HC and PolyP (0.5 vs 0.5).

## Related tools

- **iMAP** (Web server providing the automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation) — https://imap.metaboprofile.cloud/
- **IP4M** (Predecessor desktop platform whose R backend underlies iMAP's model-building workflow) — https://github.com/IP4M
- **random Forest** (Tree-ensemble classifier fitted on the selected metabolites for each group pair)
- **pROC** (R package used to compute ROC AUCs in both the train-set and test-set validation pipelines)

## Examples

```
Rscript imap_predictive_model.R --data zhu2014_serum.csv --groups CRC,PolyP,HC --methods rf,elasticnet --split 0.8 --stratify TRUE --auc-pkg pROC
```

## Evaluation signals

- auc_table.csv matches reported Table 2 values, e.g., RF HC vs. CRC 0.743 test / 1 in train; elastic net HC vs. PolyP 0.5 test / 0.5 train
- AUC in the train set exceeds AUC in the test set in all cases except elastic net between HC and PolyP
- Exactly four metabolites are selected under the criteria (Tukey P < 0.05; |log2FC| > 0.25): Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid
- Feature selection is performed on the train set only, so no test-set information leaks into the fitted models
- Twelve AUC values total (2 methods x 3 group pairs x 2 pipelines) are reported, all computed with pROC

## Limitations

- Overfitting is expected and tolerated in the train-set pipeline (RF HC vs. CRC reaches AUC 1 in train vs. 0.743 in test); only the test-set AUC reflects generalization
- Small selected feature sets (here 4 metabolites) can yield chance-level performance, as with elastic net HC vs. PolyP AUC 0.5 in both pipelines
- Raw data preprocessing and peak picking/annotation modules are not implemented in iMAP, so input must already be an annotated intensity matrix
- iMAP has acknowledged limitations that are not fully specified in the article's discussion

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] Split samples into train and test sets at 8:2 with stratified sampling; validate each fitted model twice (train and held-out test) computing ROC AUCs with pROC.: [evidence span withheld — anchor status: not_found]
- [intro] New modules for predictive model building and validation were added to iMAP's workflow.: "New modules for predictive model building and validation and correlation-based network construction and analysis have been added"
- [readme] README confirms the new module description for iMAP.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
