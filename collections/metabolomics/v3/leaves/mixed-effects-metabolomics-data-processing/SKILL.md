---
name: mixed-effects-metabolomics-data-processing
description: Use when clinical metabolomics data (e.g. GC-TOF MS or UPLC-QTOF MS feature
  tables, log2-transformed) contain subject-specific biological variation that should
  be modeled and removed prior to multivariate analyses such as PLS-DA/OPLS-DA.
when_to_use_negative:
- The data matrix has not been pre-processed and log2-transformed as described (raw
  MS signals still requiring noise reduction, peak matching and retention time correction).
- There is no subject-level metadata available to formulate the random-effects structure
  of the LMMs.
- The goal is per-metabolite differential abundance testing rather than readjusting
  signals to improve multivariate classification; note the potential introduction
  of false positives due to increased sensitivity and the article's recommendation
  of further experimental validation.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: lmm2met
  role: R package whose fitLmm function fits each metabolite to a linear mixed-effects
    model with random-intercept term (1|Id) to readjust the metabolomics matrix
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: linear model-fitting procedure on which lmm2met was implemented
- name: caret
  role: repeated k-fold (k = 4, five repeats) train/test partitioning and PLS classification
    prediction of held-out test sets
- name: mixOmics
  role: multilevel (ML) PLS-DA baseline comparison processing strategy
- name: ropls
  role: PLS-DA and OPLS-DA on M0- and LMM-processed training sets for VIP > 1 feature
    selection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:a098f2f1774e14d24bd75d941106a3d1b1f8adc00ea66cd7f9e61c057113455c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - caret
  - lme4
  - lmm2met
  - mixOmics
  - ropls
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# mixed-effects-metabolomics-data-processing

## Summary

An additional data processing step that fits a linear mixed-effects model (LMM) to each metabolite signal using patient metadata (e.g. a random-intercept term (1|Id)) to readjust individual metabolite abundances in clinical metabolomics matrices, improving downstream PLS classification performance.

## When to use

Use when clinical metabolomics data (e.g. GC-TOF MS or UPLC-QTOF MS feature tables, log2-transformed) contain subject-specific biological variation that should be modeled and removed prior to multivariate analyses such as PLS-DA/OPLS-DA.

## When NOT to use

- The data matrix has not been pre-processed and log2-transformed as described (raw MS signals still requiring noise reduction, peak matching and retention time correction).
- There is no subject-level metadata available to formulate the random-effects structure of the LMMs.
- The goal is per-metabolite differential abundance testing rather than readjusting signals to improve multivariate classification; note the potential introduction of false positives due to increased sensitivity and the article's recommendation of further experimental validation.

## Inputs

- log2-transformed metabolomics feature matrix (e.g. GC-TOF MS lung metabolite data, UPLC-QTOF MS adipose lipid data)
- patient/sample metadata (e.g. metadata of n = 39 patients with early stage lung adenocarcinoma; metadata of 59 colorectal cancer patients with VAT and SAT samples)
- random-effects grouping variable (subject Id) specifying the LMM random-intercept term (1|Id)

## Outputs

- LMM-fitted (readjusted) metabolomics matrix with reduced subject-specific variation
- discriminative metabolite lists with VIP > 1 scores per processing strategy
- averaged prediction performance table (accuracy, precision, sensitivity, specificity) across 20 train/test subsets for M0, LMM and ML strategies

## How to apply

Start with a log2-transformed metabolomics matrix (e.g. lung GC-TOF MS metabolites or adipose lipid profiles from Metabolomics Workbench PR000305/ST000390 and PR000058/ST000081) plus patient metadata to formulate the LMMs. Fit each measured metabolite to an LMM with a random-intercept term (1|Id) using the lmm2met R package's fitLmm function (implemented on lme4), producing an LMM-fitted matrix in which subject-specific variation is systematically minimized and variance per metabolite is reduced. Compare against the original (M0, no extra processing) and multilevel (ML via mixOmics) processed versions. To evaluate, partition into training/test sets with repeated k-fold cross-validation (k = 4, repeated five times, via caret), run PLS-DA/OPLS-DA (ropls, mixOmics) on M0- and LMM-processed training sets and multilevel PLS-DA (mixOmics) on ML sets, select discriminative metabolites with VIP > 1, train a PLS classification model, and average accuracy, precision, sensitivity and specificity over the 20 train/test subsets. The LMM method should improve accuracy, precision, sensitivity and specificity over M0 and ML, with ML slightly better only for specificity on the GC-TOF MS dataset (overall ~80%).

## Related tools

- **lmm2met** (R package whose fitLmm function fits each metabolite to a linear mixed-effects model with random-intercept term (1|Id) to readjust the metabolomics matrix) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (linear model-fitting procedure on which lmm2met was implemented)
- **caret** (repeated k-fold (k = 4, five repeats) train/test partitioning and PLS classification prediction of held-out test sets)
- **mixOmics** (multilevel (ML) PLS-DA baseline comparison processing strategy)
- **ropls** (PLS-DA and OPLS-DA on M0- and LMM-processed training sets for VIP > 1 feature selection)

## Examples

```
install.packages('devtools'); library(devtools); install_github('kwanjeeraw/lmm2met')
```

## Evaluation signals

- Substantial reduction in variance of each metabolite across all patients after LMM fitting, with subject-wise boxplots showing medians closer to zero and less variability within tissue type.
- PCA of LMM-fitted adipose lipid data shows better SAT/VAT separation with a 6% increase in explained variation on PC1 versus original data.
- Averaged PLS classification metrics over 20 train/test subsets show LMM superior to M0 and ML for accuracy, precision, sensitivity and specificity on both UPLC-QTOF MS and GC-TOF MS datasets (overall ~80%).
- Known exception check: ML slightly outperforms LMM for specificity on the GC-TOF MS dataset — a reproduced table should show this pattern (Figure S-4).
- Previously indistinguishable signals (e.g. cholesterol in CRC samples) become distinguishable after LMM fitting compared to the M0 PLS-DA.

## Limitations

- LMM method did not outperform the ML method for specificity on the GC-TOF MS dataset.
- Increased sensitivity carries the potential introduction of false positives.
- At the current stage the method's performance is preliminary; further experiments are recommended to validate the results.

## Evidence

- [intro] Propose the use of linear mixed-effects models (LMMs) as an additional data processing step for adjusting individual metabolite signals in metabolomics data.: "we propose the use of LMMs as an additional data processing step for adjusting an individual metabolite signal in metabolomics data."
- [methods] LMM fitting uses random intercept on subject metadata via lmm2met/lme4.: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [methods] Feature selection threshold for discriminative metabolites.: "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [results] Variance reduction outcome of LMM fitting.: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
- [discussion] Specificity exception on GC-TOF MS dataset and overall performance pattern.: "except specificity of GC-TOF MS dataset, for which the ML method was slightly better than the LMM method"
- [readme] Installation of lmm2met from GitHub via devtools.: "install_github('kwanjeeraw/lmm2met')"
