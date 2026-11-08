---
name: elastic-net-regularized-modeling
description: Use when a phenotype-grouped metabolomics feature matrix (e.g., CRC vs.
  PolyP vs. HC serum metabolite intensities) must be turned into a predictive classifier
  whose discriminatory power is quantified by ROC AUC in train and test pipelines.
when_to_use_negative:
- The input is raw spectral data — iMAP deliberately did not implement raw data preprocessing
  and peak picking/annotation modules, so feature extraction must happen elsewhere
  first
- The goal is discovery of differential metabolites rather than prediction — feature-selection
  steps (Kruskal-Wallis/Tukey/log2FC) or network analysis are the appropriate skills
  instead
- Only one phenotype group or a single-class problem exists — binary two-group comparisons
  (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP) are required for the AUC-based validation
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server whose new automatic-pipeline module performs train-test set separation,
    feature selection, and predictive model construction and validation
- name: IP4M
  role: Predecessor desktop platform; iMAP's backend R scripts (hosted under the IP4M
    GitHub organization) execute the elastic net modeling pipeline
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: random Forest
  role: Comparator classifier fit on the same selected metabolites so elastic net
    performance can be benchmarked per group pair
- name: pROC
  role: Computes the ROC AUC values used to validate each fitted model on train and
    test sets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:7cc232a3dcc2a41586f7602ab0228347a4a389ea94ffe38da056e0a25a101976
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IP4M/R-scripts-for-iMAP
  tools:
  - IP4M
  - iMAP
  - pROC
  - random Forest
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# elastic-net-regularized-modeling

## Summary

Build an L1+L2-penalized (elastic net) classifier on a selected subset of metabolite features and validate it by ROC AUC on both train and held-out test sets, as implemented in iMAP's predictive-model module alongside a random forest comparator.

## When to use

Use when a phenotype-grouped metabolomics feature matrix (e.g., CRC vs. PolyP vs. HC serum metabolite intensities) must be turned into a predictive classifier whose discriminatory power is quantified by ROC AUC in train and test pipelines.

## When NOT to use

- The input is raw spectral data — iMAP deliberately did not implement raw data preprocessing and peak picking/annotation modules, so feature extraction must happen elsewhere first
- The goal is discovery of differential metabolites rather than prediction — feature-selection steps (Kruskal-Wallis/Tukey/log2FC) or network analysis are the appropriate skills instead
- Only one phenotype group or a single-class problem exists — binary two-group comparisons (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP) are required for the AUC-based validation

## Inputs

- Targeted serum metabolite intensity matrix (234 samples: 66 CRC, 76 PolyP, 92 HC; 113 metabolites; Zhu et al., 2014)
- Group/phenotype labels per sample for train-test stratified split (8:2)
- Selected feature list from train-set filtering (e.g., 4 metabolites: Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)

## Outputs

- auc_table.csv with columns: group pair, method (RF or elastic net), AUC in test set, AUC in train set (6 AUC values per method)
- Fitted elastic net and random forest classifiers per group pair
- ROC curves computed with pROC

## How to apply

Split the metabolite intensity matrix into train and test sets (8:2 stratified random sampling as in the Zhu et al. 2014 CRC dataset: 234 samples, 66 CRC / 76 PolyP / 92 HC, 113 metabolites). Select features only from the train set using statistical criteria (Kruskal-Wallis with pairwise Tukey post-hoc P < 0.05 in at least one group pair and |log2FC| > 0.25), yielding a small panel such as Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid. Fit the elastic net (L1+L2 penalized) classifier separately for each group pair (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP), in parallel with a random forest classifier for comparison. Validate each model twice with pROC: once on the train set itself and once on the held-out test set, compiling AUC per group pair per method into a table. Judge success by comparing against the reported Table 2 values and by checking the expected overfitting signature: train AUC exceeds test AUC in all cases except elastic net between HC and PolyP (0.5 / 0.5).

## Related tools

- **iMAP** (Web server whose new automatic-pipeline module performs train-test set separation, feature selection, and predictive model construction and validation)
- **IP4M** (Predecessor desktop platform; iMAP's backend R scripts (hosted under the IP4M GitHub organization) execute the elastic net modeling pipeline) — https://github.com/IP4M/R-scripts-for-iMAP
- **random Forest** (Comparator classifier fit on the same selected metabolites so elastic net performance can be benchmarked per group pair)
- **pROC** (Computes the ROC AUC values used to validate each fitted model on train and test sets)

## Examples

```
Rscript -e 'library(glmnet); library(pROC); fit <- cv.glmnet(x_train, y_train, family="binomial", alpha=0.5); auc <- as.numeric(roc(y_test, as.numeric(predict(fit, x_test, s="lambda.min", type="response")))$auc)'
```

## Evaluation signals

- auc_table.csv matches the reported Table 2 values, e.g., RF HC vs. CRC = 0.743 test / 1 train and elastic net HC vs. PolyP = 0.5 test / 0.5 train
- AUC in the train set exceeds AUC in the test set in all cases except elastic net between HC and PolyP
- Feature selection yields exactly the four metabolites Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid under the stated criteria
- Six AUC values are recorded per method, covering all three group pairs (HC vs. CRC, CRC vs. PolyP, HC vs. PolyP) under both validation pipelines
- Train-test split preserves group proportions (8:2 stratified from 66 CRC / 76 PolyP / 92 HC)

## Limitations

- iMAP does not implement raw data preprocessing or peak picking and annotation modules, considering data security risk; data must arrive as a processed feature table
- Elastic net between HC and PolyP showed no discriminative gain (AUC 0.5 in both pipelines), so the panel does not separate polyp patients from healthy controls in this dataset
- Elastic net performance is only demonstrated on a small 4-metabolite panel from a targeted 113-metabolite serum dataset; behavior on high-dimensional or untargeted matrices is not shown here

## Evidence

- [abstract] New module for predictive model building and validation: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [intro] New modules for predictive modeling: "New modules for predictive model building and validation and correlation-based network construction and analysis have been added"
- [methods] Backend R scripts availability: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [intro] iMAP scope excludes raw preprocessing: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [readme] Flexible workflows claim: "Compared with existing platforms, iMAP has more flexible workflows and more optional criteria for differential variables selection."
