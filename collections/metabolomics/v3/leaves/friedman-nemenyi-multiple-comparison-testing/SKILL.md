---
name: friedman-nemenyi-multiple-comparison-testing
description: Use when you have repeated (e.g., 10-fold or fivefold CV) F-measure/Accuracy
  scores for multiple competing models across folds and must rank them and identify
  statistically significant pairwise winners, such as selecting among 24 classifiers
  built from peak-quality metric sets M4, M7 and M11.
when_to_use_negative:
- You have only a single train/test split or unpaired scores — Friedman–Nemenyi requires
  repeated, matched measurements (e.g., 10 CV runs) across all classifiers on the
  same folds.
- You are comparing a single classifier against a baseline rather than ranking a full
  set of ≥ 3 models — a Wilcoxon Rank Sum test (as MetaClean used for pairwise metric-set
  comparison) is the appropriate choice.
- Input is raw peak-quality metric data or an unprocessed XCMS feature table rather
  than aggregated per-fold evaluation measures — pre-compute F-measures/Accuracy first.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: scmamp
  role: R package used to run Friedman's test and Nemenyi's post-hoc test and compute
    critical differences over CV evaluation measures
- name: MetaClean
  role: Wraps the Friedman–Nemenyi comparison via getCDPlots() and getBarPlots()/summaryStats()
    to rank candidate peak-quality classifiers and export CD plots
  repo: https://CRAN.R-project.org/package=MetaClean
- name: plotrix
  role: Rendering the critical-difference plots of average classifier ranks
- name: caret
  role: Provides the training/CV machinery that produces the per-repetition evaluation
    measures fed into the Friedman–Nemenyi comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:8a6847181b1dc8f4cae93931cd87c661fa6cd439befa747a3df907c652c3d8e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - MetaClean (getCDPlots, getBarPlots, summaryStats)
  - caret
  - plotrix
  - scmamp
  - scmamp R package
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# friedman-nemenyi-multiple-comparison-testing

## Summary

Two-round statistical comparison (intra-metric set, then inter-metric set) of candidate classifiers using Friedman's test followed by Nemenyi's post-hoc test on cross-validated evaluation measures, used in MetaClean to select the global peak-quality classifier (AdaBoost with the M11 metric set).

## When to use

Use when you have repeated (e.g., 10-fold or fivefold CV) F-measure/Accuracy scores for multiple competing models across folds and must rank them and identify statistically significant pairwise winners, such as selecting among 24 classifiers built from peak-quality metric sets M4, M7 and M11.

## When NOT to use

- You have only a single train/test split or unpaired scores — Friedman–Nemenyi requires repeated, matched measurements (e.g., 10 CV runs) across all classifiers on the same folds.
- You are comparing a single classifier against a baseline rather than ranking a full set of ≥ 3 models — a Wilcoxon Rank Sum test (as MetaClean used for pairwise metric-set comparison) is the appropriate choice.
- Input is raw peak-quality metric data or an unprocessed XCMS feature table rather than aggregated per-fold evaluation measures — pre-compute F-measures/Accuracy first.

## Inputs

- Cross-validation evaluation-measure dataframe with columns Pass_FScore, Fail_FScore, Accuracy per model and per CV repetition (e.g., from MetaClean runCrossValidation/getEvaluationMeasures on the labeled ST000726 development dataset)
- List of candidate classifier identifiers (model × metric-set combinations, e.g., AdaBoost_M11, RandomForest_M7)
- Number of CV repetitions/folds (e.g., fivefold CV as in MetaClean)
- Class labels of interest (Pass = high-quality peaks, Fail = low-quality peaks)

## Outputs

- Critical-difference (CD) plot images visualizing average ranks of all candidate classifiers
- Rank matrix from Friedman's test (classifier ranks across CV repetitions)
- Summary table of Friedman–Nemenyi p-values and critical-difference values per pairwise comparison
- Named best-performing classifier per metric set (intra round) and global winner (inter round), e.g., AdaBoost_M11

## How to apply

First collect per-repetition evaluation measures (Pass FScore, Fail FScore, Accuracy) for every candidate model from the CV framework — in MetaClean, 24 classifiers formed by crossing three metric sets (Zhang et al. M4, TargetedMSQC-derived M7, and their combination M11) with eight algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, Model-Averaged Neural Network, Random Forest). Run the intra-metric set round: within each metric set, apply Friedman's test to rank the eight classifiers across the CV repetitions on the F-measure for the Pass class and for the Fail class separately, then apply Nemenyi's post-hoc test to find significant pairwise differences. Advance the best-performing classifier per metric set to the inter-metric set round and repeat Friedman–Nemenyi testing across metric sets. Use the scmamp R package (v0.2.55) as implemented in MetaClean's getCDPlots() to compute critical difference (CD) values and rank matrices, and render CD plots that visualize average ranks following Demsar (2006). Conclude that a classifier is the global winner only when it is ranked first and its pairwise Nemenyi p-values versus rivals fall below the critical-difference threshold on both Pass and Fail F-measures (AdaBoost_M11: Pass p = 0.001 and 0.005; Fail p = 0.0023 and 0.0023).

## Related tools

- **scmamp** (R package used to run Friedman's test and Nemenyi's post-hoc test and compute critical differences over CV evaluation measures)
- **MetaClean** (Wraps the Friedman–Nemenyi comparison via getCDPlots() and getBarPlots()/summaryStats() to rank candidate peak-quality classifiers and export CD plots) — https://CRAN.R-project.org/package=MetaClean
- **plotrix** (Rendering the critical-difference plots of average classifier ranks)
- **caret** (Provides the training/CV machinery that produces the per-repetition evaluation measures fed into the Friedman–Nemenyi comparison)

## Examples

```
Rscript -e 'library(MetaClean); cd <- getCDPlots(evalMeasures, "classifier", c("Pass_FScore","Fail_FScore"))'
```

## Evaluation signals

- Rank matrix places AdaBoost_M11 at rank 1 (or statistically tied) for both Pass and Fail F-measure in the inter-metric comparison.
- Nemenyi pairwise p-values for AdaBoost_M11 versus rivals meet the reported values: Pass F-measure p = 0.001 and 0.005; Fail F-measure p = 0.0023 and 0.0023.
- The computed critical difference value is consistent with the number of classifiers (24 total; 8 per metric set in the intra round) and number of CV repetitions (10) used.
- CD plot image is produced for each round (intra-metric set and inter-metric set) and for both Pass and Fail classes.
- Every candidate classifier appears exactly once per round in the rank matrix; no model is dropped before ranking.

## Limitations

- Friedman–Nemenyi tests require the same folds/repetitions for all classifiers; results are sensitive to the choice of evaluation measure (Pass vs. Fail F-measure) and can disagree between classes.
- MetaClean observed only moderately significant differences between metric-set pairs via Wilcoxon Rank Sum tests, so ties or near-ties among top classifiers are common and the CD plot must be read with the critical-difference bar in mind.
- The comparison is only as good as the CV evaluation measures feeding it; conservative classifier behavior (e.g., worse Fail F-measure on Test 1/2 due to fewer low-quality peaks in development) can distort rankings when applied to external datasets.

## Evidence
