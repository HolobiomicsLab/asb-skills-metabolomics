---
name: critical-difference-plot-visualization
description: Use when several classifiers (e.g. three metric sets × eight algorithms
  such as Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear,
  AdaBoost, Model-Averaged Neural Network, Random Forest) have been compared over
  repeated cross-validation runs and you need a rank-based.
when_to_use_negative:
- You have only two classifiers to compare — a Wilcoxon Rank Sum test / scatter-plot
  comparison (as the article uses for metric-set pairs) is more appropriate than Friedman–Nemenyi
  CD plots.
- Input is a labeled feature table or raw LC–MS data rather than already-collected
  cross-validation performance statistics — peak quality classification must be done
  first (MetaClean's runCrossValidation) before CD plots can be rendered.
- You need an absolute performance ranking for deployment rather than a statistically
  justified choice among candidates — CD plots show relative ranks and significance,
  not accuracy/F-measure magnitudes.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: MetaClean
  role: Provides getCDPlots (with getBarPlots and summaryStats) to compute Friedman–Nemenyi
    statistics and render the critical-difference plots over cross-validation evaluation
    measures
  repo: https://CRAN.R-project.org/package=MetaClean
- name: scmamp
  role: R package (version 0.2.55) used to run Friedman's test followed by Nemenyi's
    test and generate CD plots/rank matrices as described by Demsar (2006)
- name: caret
  role: Trains the candidate classifiers whose CV evaluation measures feed the Friedman–Nemenyi
    comparison (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear,
    AdaBoost, Model-Averaged Neural Network, Random Forest)
- name: XCMS
  role: Upstream pre-processing (peak-picking, retention time correction, grouping)
    that produces the labeled peak data underlying the CV evaluation measures
- name: plotrix
  role: Rendering/rank-plot support referenced alongside scmamp for visualizing average
    ranks of the candidate classifiers
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:a9857f6ff775e07d50c2b9e0a3b7aecd6daf460f1f95d279de0ad398d4f03c88
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
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# critical-difference-plot-visualization

## Summary

Visualize the outcome of Friedman–Nemenyi statistical comparison of multiple machine learning classifiers as critical-difference (CD) plots that rank candidates (e.g. 24 classifiers from three peak-quality metric sets and eight algorithms) by average F-measure rank and flag statistically distinguishable groups.

## When to use

Use when several classifiers (e.g. three metric sets × eight algorithms such as Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged Neural Network, Random Forest) have been compared over repeated cross-validation runs and you need a rank-based, statistically backed visual to pick the global best-performing classifier.

## When NOT to use

- You have only two classifiers to compare — a Wilcoxon Rank Sum test / scatter-plot comparison (as the article uses for metric-set pairs) is more appropriate than Friedman–Nemenyi CD plots.
- Input is a labeled feature table or raw LC–MS data rather than already-collected cross-validation performance statistics — peak quality classification must be done first (MetaClean's runCrossValidation) before CD plots can be rendered.
- You need an absolute performance ranking for deployment rather than a statistically justified choice among candidates — CD plots show relative ranks and significance, not accuracy/F-measure magnitudes.

## Inputs

- Cross-validation evaluation-measure dataframe (per classifier and CV repetition: Pass_FScore, Fail_FScore, Accuracy) covering the 24 candidate classifiers (metric sets M4, M7, M11 × eight algorithms) trained on a labeled peak-quality dataset (e.g. Metabolomics Workbench ST000726)

## Outputs

- Critical-difference (CD) plot image(s) showing average ranks of all 24 candidate classifiers for Pass F-measure and Fail F-measure
- Rank matrix from Friedman's test
- Summary table of Friedman–Nemenyi p-values and critical differences

## How to apply

Load the cross-validation evaluation-measure dataframe containing per-run, per-classifier measures (e.g. Pass_FScore, Fail_FScore, Accuracy over ten CV runs) and apply Friedman's test to the performance statistics, followed by Nemenyi's post-hoc test as described by Demsar (2006), using the scmamp R package (version 0.2.55) via MetaClean's getCDPlots. Run two rounds of comparison: an intra-metric set round (comparing the eight classifiers within each metric set M4, M7, M11 on F-measure for the Pass class and F-measure for the Fail class) and an inter-metric set round (comparing the best classifier from each metric set across all sets). Compute the critical difference value and pairwise p-values, then render CD plots that display the average rank of each candidate classifier together with the critical-difference bar. Correct application is verified when the CD plot and rank matrix show AdaBoost with the M11 metric set at rank 1 with the reported significance (Pass F-measure p = 0.001 and 0.005; Fail F-measure p = 0.0023 and 0.0023) matching the paper's finding that AdaBoost_M11 significantly outperforms the other candidate classifiers.

## Related tools

- **MetaClean** (Provides getCDPlots (with getBarPlots and summaryStats) to compute Friedman–Nemenyi statistics and render the critical-difference plots over cross-validation evaluation measures) — https://CRAN.R-project.org/package=MetaClean
- **scmamp** (R package (version 0.2.55) used to run Friedman's test followed by Nemenyi's test and generate CD plots/rank matrices as described by Demsar (2006))
- **caret** (Trains the candidate classifiers whose CV evaluation measures feed the Friedman–Nemenyi comparison (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged Neural Network, Random Forest))
- **XCMS** (Upstream pre-processing (peak-picking, retention time correction, grouping) that produces the labeled peak data underlying the CV evaluation measures)
- **plotrix** (Rendering/rank-plot support referenced alongside scmamp for visualizing average ranks of the candidate classifiers)

## Examples

```
Rscript -e "library(MetaClean); getCDPlots(getEvaluationMeasures(runCrossValidation(...)))"
```

## Evaluation signals

- The inter-metric round CD plot places AdaBoost_M11 at rank 1 for both Pass F-measure and Fail F-measure, matching the reported Friedman–Nemenyi p-values (Pass: 0.001 and 0.005; Fail: 0.0023 and 0.0023).
- The rank matrix from Friedman's test contains one ranked row per CV run across the eight classifiers per metric set (10 CV runs from the fivefold CV).
- A single critical-difference value is computed per round and is consistent across all classifiers in the CD plot (the Nemenyi critical difference depends only on the number of candidates and CV runs).
- The exported summary table reports pairwise p-values and the critical difference for each classifier comparison, in addition to the visual CD plot.
- Intra-metric set round first identifies the best classifier per metric set (AdaBoost for M4, Random Forest for M7, AdaBoost for M11), and the inter-metric round compares only those three candidates.

## Limitations

- Friedman–Nemenyi testing requires the full matrix of per-run performance statistics; the paper's supplementary scatter plots do not report exact performance metric values for the 24 candidate classifiers, so CD plots can be reconstructed only from re-run cross-validation output, not from published numbers.
- CD plots rank candidates by average F-measure rank and significance; they do not convey absolute Accuracy or F-measure magnitudes, which in the article were reported qualitatively (e.g. Test 1 'performed well in terms of Accuracy and Pass and Fail F-measures').
- The upstream classifier performance is affected by class imbalance — the article notes the classifier performed worse for the low-quality (Fail) class partly because the development sets contained fewer low-quality peaks — so Fail-class CD rankings may be less stable than Pass-class rankings.

## Evidence
