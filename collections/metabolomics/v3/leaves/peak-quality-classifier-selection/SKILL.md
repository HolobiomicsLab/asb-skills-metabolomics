---
name: peak-quality-classifier-selection
description: Use when you have cross-validation performance scores (e.g. Pass/Fail
  F-measure, Accuracy) for multiple candidate classifiers (algorithm × peak-quality
  metric set combinations) and must pick a single classifier with statistical justification
  rather than eyeballing averages.
when_to_use_negative:
- You only have a single classifier's scores and no competing candidates to rank —
  there is nothing for Friedman/Nemenyi to compare.
- You are evaluating a trained classifier on a held-out validation dataset (e.g. Test
  1/Test 2); this skill is for selecting among candidates during training, not for
  final external validation reporting.
- Your performance data are not paired over shared CV folds/runs, so ranked comparisons
  across candidates (the assumption behind Friedman's test) do not hold.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: MetaClean
  role: Provides runCrossValidation to produce the CV evaluation-measure dataframe
    that feeds the statistical comparison, and getCDPlots to run Friedman/Nemenyi
    testing and render critical-difference plots
  repo: https://CRAN.R-project.org/package=MetaClean
- name: scmamp
  role: R package (version 0.2.55) used to perform Friedman's test and Nemenyi's post-hoc
    multiple-comparison test and to build rank matrices and CD plots
- name: XCMS
  role: Pre-processed the raw LC–MS data (peak-picking, retention time correction,
    grouping) that generated the peaks later labeled Pass/Fail and used to train the
    candidate classifiers
- name: caret
  role: Used to train the eight candidate classification algorithms (Decision Tree,
    Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged
    Neural Network, Random Forest) over the metric sets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:9fd38a1d802805db25a3907a464d7b15a4c0347914e86f4cbd775c40f7ba784d
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# peak-quality-classifier-selection

## Summary

Selects the global best-performing peak-quality classifier by statistically comparing candidate algorithm–metric-set combinations using Friedman's test followed by Nemenyi's post-hoc test, as implemented in MetaClean for untargeted LC–MS metabolomics.

## When to use

Use when you have cross-validation performance scores (e.g. Pass/Fail F-measure, Accuracy) for multiple candidate classifiers (algorithm × peak-quality metric set combinations) and must pick a single best classifier with statistical justification rather than eyeballing averages.

## When NOT to use

- You only have a single classifier's scores and no competing candidates to rank — there is nothing for Friedman/Nemenyi to compare.
- You are evaluating a trained classifier on a held-out validation dataset (e.g. Test 1/Test 2); this skill is for selecting among candidates during training, not for final external validation reporting.
- Your performance data are not paired over shared CV folds/runs, so ranked comparisons across candidates (the assumption behind Friedman's test) do not hold.

## Inputs

- Cross-validation evaluation-measure dataframe (Pass_FScore, Fail_FScore, Accuracy per model per CV repetition) from MetaClean's runCrossValidation over candidate classifiers trained on the labeled development dataset (ST000726)
- List of candidate classifiers defined as algorithm × metric-set combinations (three metric sets × eight algorithms = 24 candidates)

## Outputs

- Critical-difference (CD) plots showing average ranks of all candidate classifiers for Pass and Fail F-measures
- Rank matrices from Friedman's test
- Summary table of Friedman–Nemenyi p-values and critical-difference values per pairwise comparison
- Identification of the single global best classifier (here: AdaBoost with the M11 metric set)

## How to apply

Train all candidate classifiers — combinations of three metric sets (M4, M7, M11) and eight algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged Neural Network, Random Forest) — over ten runs of the fivefold cross-validation procedure on the labeled development dataset (Metabolomics Workbench ST000726), collecting Pass FScore, Fail FScore, and Accuracy per run. Then run two rounds of comparison: first an intra-metric set round, applying Friedman's test to ranks of the eight algorithms within each metric set across the ten CV runs, followed by Nemenyi's multiple-comparison post-hoc test (Demsar 2006) on Pass F-measure and Fail F-measure, to select the best classifier per metric set; then an inter-metric set round comparing the best-performing classifier from each metric set. Execute the tests with the scmamp R package (version 0.2.55) via MetaClean's getCDPlots to generate critical-difference (CD) plots and rank matrices visualizing average ranks of all candidate classifiers. Decision point: the global best classifier is the one ranked first (or statistically tied for first) on both Pass and Fail F-measures — in this work that was AdaBoost paired with the M11 metric set, significant with Nemenyi p-values of 0.001/0.005 (Pass) and 0.0023/0.0023 (Fail).

## Related tools

- **MetaClean** (Provides runCrossValidation to produce the CV evaluation-measure dataframe that feeds the statistical comparison, and getCDPlots to run Friedman/Nemenyi testing and render critical-difference plots) — https://CRAN.R-project.org/package=MetaClean
- **scmamp** (R package (version 0.2.55) used to perform Friedman's test and Nemenyi's post-hoc multiple-comparison test and to build rank matrices and CD plots)
- **XCMS** (Pre-processed the raw LC–MS data (peak-picking, retention time correction, grouping) that generated the peaks later labeled Pass/Fail and used to train the candidate classifiers)
- **caret** (Used to train the eight candidate classification algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged Neural Network, Random Forest) over the metric sets)

## Examples

```
library(MetaClean); eval_df <- runCrossValidation(...); getCDPlots(eval_df, metrics = c("Pass_FScore", "Fail_FScore"))
```

## Evaluation signals

- The selected classifier ranks first (or is statistically tied for first) in the critical-difference plot for BOTH the Pass F-measure and Fail F-measure inter-metric comparisons.
- Nemenyi post-hoc p-values for the best classifier against rivals are below 0.05 on both Pass and Fail F-measures; for the reference result these were 0.001/0.005 (Pass) and 0.0023/0.0023 (Fail).
- The intra-metric set round yields exactly one best classifier per metric set (AdaBoost for M4, Random Forest for M7, AdaBoost for M11), and the inter-metric round compares exactly these three.
- Friedman's test is computed across the same ten CV runs for every candidate classifier (paired ranks over identical folds), and the rank matrix dimensions equal the number of candidates compared.
- The exported CD plots, rank matrices, and p-value summary table reproduce the qualitative ordering of candidates shown in the paper's Supplementary scatter plots.

## Limitations

- Wilcoxon Rank Sum tests showed only moderately significant differences between metric sets, so metric-set choice matters less than algorithm choice for distinguishing candidates.
- The comparison relies on paired CV performance over ten runs; results are sensitive to the number of CV repetitions and the size and class balance of the labeled development set.
- This procedure identifies the best classifier on the development data distribution; the paper notes the selected classifier later performed worse on the Fail class for external test sets, so selection significance does not guarantee transfer performance.
- The final selection here is only valid within the same analytical context — ionization mode, analytical platform and method, and biological matrix affect peak quality and thus the relative performance of metric sets.

## Evidence

- [results] Using Friedman's and Nemenyi's tests, AdaBoost with the M11 metric set performed significantly better than other candidate classifiers in F-measure for both Pass (p = 0.001 and 0.005) and Fail (p = 0.0023 and 0.0023) classes.: [evidence span withheld — anchor status: not_found]
