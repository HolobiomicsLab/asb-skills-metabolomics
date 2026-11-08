---
name: cross-validation-evaluation-measure-analysis
description: Use when multiple peak-quality metric sets (e.g., M4, M7, M11) must be
  paired with diverse ML algorithms and statistically compared via cross-validation
  to identify a -performing classifier.
when_to_use_negative:
- When only a single classifier needs to be assessed on one metric set without needing
  a statistical comparison across multiple candidate classifiers.
- When the goal is filtering peaks by a traditional RSD < 30% threshold rather than
  comparing ML classifiers' F-measures.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: MetaClean
  role: Provides runCrossValidation, getEvaluationMeasures, and getCDPlots to run
    the CV framework, collect Accuracy/FScore measures, and render critical-difference
    plots for Friedman/Nemenyi comparison.
  repo: https://CRAN.R-project.org/package=MetaClean
- name: scmamp
  role: R package used via getCDPlots for Friedman's test and Nemenyi's post-hoc test
    on the cross-validation performance statistics.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:eae89045977004318efdd9add6c3b0ab9903d7cf19ee4ce9d977b85efefee0d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - MetaClean (getCDPlots, getBarPlots, summaryStats)
  - plotrix
  - scmamp
  - scmamp R package
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-validation-evaluation-measure-analysis

## Summary

This skill applies fivefold cross-validation with classification evaluation measures (Accuracy, Pass/Fail F-measure) and Friedman/Nemenyi statistical comparison to rank candidate peak-quality classifiers for untargeted LC-MS metabolomics.

## When to use

Use when multiple peak-quality metric sets (e.g., M4, M7, M11) must be paired with diverse ML algorithms and statistically compared via cross-validation to identify a best-performing classifier.

## When NOT to use

- When only a single classifier needs to be assessed on one metric set without needing a statistical comparison across multiple candidate classifiers.
- When the goal is filtering peaks by a traditional RSD < 30% threshold rather than comparing ML classifiers' F-measures.

## Inputs

- labeled peak-quality evaluation dataframe from ST000726 development dataset (Pass_FScore, Fail_FScore, Accuracy per model and per CV repetition)
- candidate classifier definitions spanning three metric sets (M4, M7, M11) × eight classification algorithms
- ten cross-validation repetition results per classifier

## Outputs

- Critical-difference (CD) plot images showing average ranks of all 24 candidate classifiers
- Rank matrices from Friedman's test per metric set and class
- Summary table of Friedman-Nemenyi p-values and critical differences for pairwise classifier comparisons

## How to apply

For each of the 24 candidate classifiers (three metric sets × eight algorithms: Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged Neural Network, Random Forest), run a rigorous fivefold cross-validation procedure on the labeled ST000726 development dataset using MetaClean's runCrossValidation and getEvaluationMeasures() to collect Accuracy, Pass FScore, and Fail FScore per CV repetition. Apply Friedman's statistical test followed by Nemenyi's multiple-comparison post-hoc test to the collected performance statistics in two rounds: first an intra-metric set round comparing the eight classifiers within each metric set, then an inter-metric set round comparing the best classifier from each metric set. Use the scmamp R package (v0.2.55, via getCDPlots) on the F-measure for the Pass class and F-measure for the Fail class to compute critical differences, rank matrices, and p-values, rendering CD plots that visualize average ranks of all 24 classifiers. Verify that the winning classifier (AdaBoost_M11) is ranked first or statistically tied for first for both Pass and Fail F-measures before exporting CD plots, rank matrices, and the summary p-value table.

## Related tools

- **MetaClean** (Provides runCrossValidation, getEvaluationMeasures, and getCDPlots to run the CV framework, collect Accuracy/FScore measures, and render critical-difference plots for Friedman/Nemenyi comparison.) — https://CRAN.R-project.org/package=MetaClean
- **scmamp** (R package used via getCDPlots for Friedman's test and Nemenyi's post-hoc test on the cross-validation performance statistics.)

## Examples

```
Rscript runCrossValidation.R --dataset ST000726 --metricSets M4,M7,M11 --algorithms DT,LR,NB,NN,SVML,AdaBoost,MANet,RF
```

## Evaluation signals

- Friedman-Nemenyi test yields p = 0.001 and 0.005 for Pass F-measure and p = 0.0023 and 0.0023 for Fail F-measure for AdaBoost_M11 against rivals
- CD plot places AdaBoost_M11 at rank 1 (or statistically tied) in the inter-metric comparison for both Pass and Fail F-measures
- Evaluation-measure dataframe contains Accuracy, Pass_FScore, and Fail_FScore columns for all 24 candidate classifiers across ten CV runs

## Limitations

- MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs
- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one (Pass) for Test 1 and 2, partly attributed to the smaller number of low-quality peaks in the classifier development sets
- Even with manual assessment recommended, the paper relies on automated flagging; ionization mode, analytical platform and method, and biological matrix will affect peak shape and size, and ultimately the quality of integrations

## Evidence

- [methods] we applied Friedman's statistical test, followed by Nemenyi's test, to the collected performance statistics (Demsar 2006): [evidence span withheld — anchor status: not_found]
