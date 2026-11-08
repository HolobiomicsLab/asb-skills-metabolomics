---
name: roc-auc-model-validation
description: Use when a labeled metabolomics feature table (e.g. 234 CRC/PolyP/HC
  serum samples) must be split into train/test sets, modeled with RF and elastic net,
  and its discriminative power reported as ROC AUC values.
when_to_use_negative:
- Raw mass-spectrometry pre-processing is needed — iMAP does not implement raw data
  preprocessing or peak picking and annotation modules
- Only differential metabolite screening is required (Kruskal-Wallis/Tukey/log2FC)
  with no predictive modeling or AUC estimation
- The dataset has unlabeled samples, so stratified train-test separation and supervised
  classification cannot be performed
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server providing the automatic pipeline for train-test set separation,
    feature selection, and predictive model construction and validation
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: IP4M
  role: Desktop predecessor platform whose metabolomics analysis workflows iMAP extends;
    backend R scripts for iMAP are hosted under its repository
  repo: https://github.com/IP4M
- name: random Forest
  role: Classifier fitted on the four selected metabolites for each group pair to
    estimate train and test AUC
- name: pROC
  role: R package used to compute ROC AUC values for both validation pipelines
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:ea3b74611b6d98d632f703eea3b14b54521147a9d670573d7b7ab4ed56395f34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IP4M/R-scripts-for-iMAP
  tools:
  - IP4M
  - iMAP
  - pROC
  - random Forest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# roc-auc-model-validation

## Summary

Build and validate predictive classifiers (random forest and elastic net) on selected metabolites using iMAP's automatic pipeline, then quantify discriminative performance with ROC AUC (pROC) on both train and held-out test sets.

## When to use

Use when a labeled metabolomics feature table (e.g. 234 CRC/PolyP/HC serum samples) must be split into train/test sets, modeled with RF and elastic net, and its discriminative power reported as ROC AUC values.

## When NOT to use

- Raw mass-spectrometry pre-processing is needed — iMAP does not implement raw data preprocessing or peak picking and annotation modules
- Only differential metabolite screening is required (Kruskal-Wallis/Tukey/log2FC) with no predictive modeling or AUC estimation
- The dataset has unlabeled samples, so stratified train-test separation and supervised classification cannot be performed

## Inputs

- Targeted serum metabolite intensity matrix: 234 samples (66 CRC, 76 PolyP, 92 HC) x 113 metabolites (Zhu et al., 2014)
- Sample class labels for the three groups (CRC, PolyP, HC)
- Feature-selection criteria: pairwise Tukey post-hoc P < 0.05 after Kruskal-Wallis and |log2FC| > 0.25

## Outputs

- auc_table.csv with columns: group pair, method (RF / elastic net), AUC in test set, AUC in train set (six rows per method)
- Fitted random forest and elastic-net classifiers per group pair
- ROC objects/AUC values computed with pROC

## How to apply

Load the targeted serum metabolite intensity matrix (234 samples: 66 CRC, 76 PolyP, 92 HC; 113 metabolites; Zhu et al., 2014) and split samples into train and test sets at an 8:2 ratio using stratified random sampling. Select features on the train set only, using the iMAP criteria: pairwise Tukey post-hoc P < 0.05 in at least one group pair after a Kruskal-Wallis test, plus |log2FC| > 0.25 for both CRC vs. PolyP and CRC vs. HC — this yields the four metabolites Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid. Fit a random forest classifier and an L1+L2 elastic-net classifier separately for HC vs. CRC, CRC vs. PolyP, and HC vs. PolyP. Validate each model twice — on the train set itself (the paper's pipeline) and on the held-out test set — computing ROC AUCs with pROC, then compile the six AUC values per method into auc_table.csv recording group pair, method, test-set AUC, and train-set AUC.

## Related tools

- **iMAP** (Web server providing the automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation) — https://github.com/IP4M/R-scripts-for-iMAP
- **IP4M** (Desktop predecessor platform whose metabolomics analysis workflows iMAP extends; backend R scripts for iMAP are hosted under its repository) — https://github.com/IP4M
- **random Forest** (Classifier fitted on the four selected metabolites for each group pair to estimate train and test AUC)
- **pROC** (R package used to compute ROC AUC values for both validation pipelines)

## Examples

```
library(pROC); auc_obj <- auc(roc(labels, predictions))
```

## Evaluation signals

- auc_table.csv reproduces the reported Table 2 values (e.g., RF HC vs. CRC = 0.743 test / 1 train; elastic net HC vs. PolyP = 0.5 test / 0.5 train)
- AUC on the train set exceeds AUC on the test set in all cases except elastic net between HC and PolyP
- Feature selection retains exactly Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid under the Tukey P < 0.05 and |log2FC| > 0.25 criteria
- Train/test split is 8:2 with stratified random sampling, preserving the 66/76/92 CRC/PolyP/HC group composition
- Each method yields six AUC values, one per group pair (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP) per validation pipeline (train, test)

## Limitations

- Feature selection and model fitting must use the train set only; selecting features on the full dataset would leak information and inflate test AUC
- AUC on the train set can reach 1 (overfitting signal), so train-set AUC alone is not a meaningful performance metric
- The workflow was demonstrated on a single targeted serum metabolomics dataset (113 metabolites); performance on higher-dimensional untargeted matrices is not established here
- Elastic net HC vs. PolyP AUC of 0.5 shows some phenotype pairs may not be discriminable from these metabolites

## Evidence

- [abstract] New module for predictive model building and validation: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] Demonstration dataset description: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [intro] Motivation for new predictive-modeling modules: "New modules for predictive model building and validation and correlation-based network construction and analysis have been added"
- [readme] Flexible workflows supporting the pipeline: "Compared with existing platforms, iMAP has more flexible workflows and more optional criteria for differential variables selection."
