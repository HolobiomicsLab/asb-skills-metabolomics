---
name: classifier-performance-statistical-comparison
description: Use when several classification algorithms or metric sets have been evaluated
  over repeated CV runs and a statistically defensible global winner must be chosen
  from correlated F-measure/Accuracy distributions.
when_to_use_negative:
- Comparing only two classifiers or methods on independent samples — a Wilcoxon Rank
  Sum test or paired t-test is the appropriate tool, not Friedman–Nemenyi
- The CV evaluation measures are not yet computed (no repeated performance statistics
  exist to rank); run the cross-validation first
- Candidate classifiers were trained and tested on non-overlapping datasets so their
  per-run scores are not paired across a common set of CV runs — the ranked repeated-measures
  design of Friedman's test does not apply
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaClean
  role: Provides runCrossValidation/getEvaluationMeasures to collect per-run CV statistics
    and getCDPlots to generate critical-difference plots for the Friedman–Nemenyi
    comparison
  repo: https://CRAN.R-project.org/package=MetaClean
- name: scmamp
  role: R package (version 0.2.55) used to execute Friedman's test followed by Nemenyi's
    post-hoc test and compute rank matrices and critical differences
- name: caret
  role: Used to train and cross-validate the eight candidate classification algorithms
    over the three metric sets
- name: XCMS
  role: Pre-processing of raw LC–MS data (peak-picking, retention time correction,
    grouping) that yields the feature table fed to the classifiers being compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:edd65e299971625674259734efdfa23ae72c0a4c135cfc79869526c8688e4d1c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - MetaClean (getCDPlots, getBarPlots, summaryStats)
  - XCMS
  - caret
  - plotrix
  - scmamp
  - scmamp R package
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# classifier-performance-statistical-comparison

## Summary

Selects a global best-performing classifier from a pool of candidates by comparing their cross-validation performance with Friedman's test followed by Nemenyi's post-hoc test (Demsar 2006), as applied in MetaClean to 24 candidate peak-quality classifiers on untargeted LC–MS metabolomics data.

## When to use

Use when several classification algorithms or metric sets have been evaluated over repeated CV runs and a statistically defensible global winner must be chosen from correlated F-measure/Accuracy distributions.

## When NOT to use

- Comparing only two classifiers or methods on independent samples — a Wilcoxon Rank Sum test or paired t-test is the appropriate tool, not Friedman–Nemenyi
- The CV evaluation measures are not yet computed (no repeated performance statistics exist to rank); run the cross-validation first
- Candidate classifiers were trained and tested on non-overlapping datasets so their per-run scores are not paired across a common set of CV runs — the ranked repeated-measures design of Friedman's test does not apply

## Inputs

- Evaluation-measure dataframe from repeated cross-validation, with columns such as Pass_FScore, Fail_FScore, and Accuracy indexed by classifier and CV run (e.g. output of MetaClean runCrossValidation/getEvaluationMeasures)
- List or matrix of candidate classifiers (e.g. 24 combinations of metric sets M4/M7/M11 and eight machine learning algorithms)
- Labeled development dataset (e.g. Metabolomics Workbench ST000726) used for the CV runs

## Outputs

- Critical-difference (CD) plot images showing average ranks of all candidate classifiers for Pass and Fail F-measures
- Rank matrices from Friedman's test
- Summary table of Friedman–Nemenyi p-values and critical differences for each pairwise classifier comparison
- Designation of the global best-performing classifier (e.g. AdaBoost_M11) for downstream deployment

## How to apply

Collect per-run evaluation measures (Pass_FScore, Fail_FScore, Accuracy) from a repeated cross-validation scheme — in MetaClean, a fivefold CV applied to 24 candidate classifiers formed by pairing three peak-quality metric sets (M4, M7, M11 from Zhang et al. 2014 and TargetedMSQC-derived metrics) with eight algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged Neural Network, Random Forest). Run the comparison in two rounds: an intra-metric set round, applying Friedman's test on the ranks of the eight algorithms across the ten CV runs for each metric set, and an inter-metric set round comparing the best algorithm from each metric set. Follow Friedman's test with Nemenyi's multiple-comparison post-hoc test, computing critical differences and pairwise p-values and rendering critical-difference (CD) plots, e.g. with the scmamp R package (version 0.2.55) via MetaClean's getCDPlots. Declare a global winner only when one candidate is ranked first (or statistically tied) for both the Pass and Fail F-measures; in the MetaClean study this was AdaBoost with the M11 metric set (Pass Friedman-Nemenyi p = 0.001 and 0.005; Fail p = 0.0023 and 0.0023). Export CD plot images, rank matrices, and a summary table of p-values and critical differences so the selection is auditable.

## Related tools

- **MetaClean** (Provides runCrossValidation/getEvaluationMeasures to collect per-run CV statistics and getCDPlots to generate critical-difference plots for the Friedman–Nemenyi comparison) — https://CRAN.R-project.org/package=MetaClean
- **scmamp** (R package (version 0.2.55) used to execute Friedman's test followed by Nemenyi's post-hoc test and compute rank matrices and critical differences)
- **caret** (Used to train and cross-validate the eight candidate classification algorithms over the three metric sets)
- **XCMS** (Pre-processing of raw LC–MS data (peak-picking, retention time correction, grouping) that yields the feature table fed to the classifiers being compared)

## Examples

```
Rscript -e 'library(MetaClean); library(scmamp); eval_df <- getEvaluationMeasures(runCrossValidation(...)); res <- friedmanTest(eval_df); nemenyi <- nemenyiTest(res); plotCD(nemenyi)'
```

## Evaluation signals

- The global winner must be ranked first (or statistically tied) for both the Pass and Fail F-measures in the inter-metric-set Friedman–Nemenyi comparison — matching AdaBoost_M11 in the reference study
- Reported pairwise p-values should match the reference result for AdaBoost_M11 vs. rivals: Pass F-measure p = 0.001 and 0.005, Fail F-measure p = 0.0023 and 0.0023
- CD plot images, rank matrices, and a p-value/critical-difference summary table must be exported to disk for auditability
- The intra-metric set round should be completed before the inter-metric set round; the best algorithm per metric set must be carried forward into the second round
- Ranking must be computed across the same repeated CV runs (e.g. ten runs) so that the repeated-measures structure required by Friedman's test is respected

## Limitations

- The statistical comparison reflects performance on the development dataset (and CV runs); generalization to independent test sets is evaluated separately, and MetaClean's classifier was conservative, performing worse for the low-quality (Fail) class than the high-quality (Pass) class on Test 1 and Test 2
- Only moderately significant differences between metric sets were observed with Wilcoxon Rank Sum tests, so the comparison cannot always cleanly separate metric sets by itself
- The underlying evaluation measures depend on XCMS pre-processing and the labeled training data; the skill does not validate pre-processing quality itself
- MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs, which constrains what classifiers/pre-processing can feed the comparison

## Evidence
