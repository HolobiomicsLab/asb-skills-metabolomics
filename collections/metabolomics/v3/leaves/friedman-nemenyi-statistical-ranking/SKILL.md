---
name: friedman-nemenyi-statistical-ranking
description: Use when comparing multiple classifier variants (e.g. combinations of
  metric sets and learning algorithms) evaluated under cross-validation, and you need
  a statistically defensible selection of the performer rather than eyeballing mean
  F-measures.
when_to_use_negative:
- Only two classifiers are being compared — a Wilcoxon Rank Sum (or paired) test per
  metric-pair is what the paper used for pairwise metric-set differences, not Friedman-Nemenyi
- Performance estimates are not paired across folds/repetitions (independent samples),
  since Friedman's test assumes a repeated-measures/blocked design
- You need per-metric-set pairwise significance between metric sets rather than a
  global ranking — the paper found only moderately significant Wilcoxon p-values between
  metric pairs, indicating the two-round Nemenyi approach is overkill for that narrow
  question
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: scmamp
  role: R package used to run Friedman's test followed by Nemenyi's post-hoc test
    on the collected cross-validation performance statistics and to compute critical-difference
    plots
- name: caret
  role: Provides the eight classification algorithms whose cross-validated performance
    is ranked by the Friedman-Nemenyi procedure
- name: MetaClean
  role: Encapsulates the full pipeline including getEvaluationMeasures() and getCDPlots()
    that feed the statistical ranking rounds
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:ea4b30581187df07d88ca2606b92f200580f4eb10e456a3a0aaf699340f70804
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - caret
  - scmamp
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# friedman-nemenyi-statistical-ranking

## Summary

A two-round statistical comparison procedure using Friedman's test followed by Nemenyi's post-hoc test (via the scmamp package) to rank candidate classifiers across metric sets and select the global best-performing model, as implemented in MetaClean to choose AdaBoost with the M11 peak-quality metric set.

## When to use

Use when comparing multiple classifier variants (e.g. combinations of metric sets and learning algorithms) evaluated under cross-validation, and you need a statistically defensible selection of the best performer rather than eyeballing mean F-measures.

## When NOT to use

- Only two classifiers are being compared — a Wilcoxon Rank Sum (or paired) test per metric-pair is what the paper used for pairwise metric-set differences, not Friedman-Nemenyi
- Performance estimates are not paired across folds/repetitions (independent samples), since Friedman's test assumes a repeated-measures/blocked design
- You need per-metric-set pairwise significance between metric sets rather than a global ranking — the paper found only moderately significant Wilcoxon p-values between metric pairs, indicating the two-round Nemenyi approach is overkill for that narrow question

## Inputs

- Cross-validation performance table: per-classifier Accuracy and F-measure for Pass and Fail classes across folds and repetitions (e.g. from getEvaluationMeasures() on 24 classifiers = 3 metric sets × 8 algorithms)
- List of candidate classifier identifiers to be ranked
- Fold/repetition identifiers used as blocking factor for the repeated-measures design

## Outputs

- Ranked ordering of candidate classifiers with Friedman test statistic and Nemenyi post-hoc p-values (e.g. AdaBoost/M11 significantly better than others on Pass and Fail F-measure)
- Critical-difference plot per comparison round (getCDPlots(), Supplementary Fig. 3 scatter plots of pairwise classifier performance)
- Selected global classifier specification (metric set + algorithm), used downstream as input to trainClassifier()

## How to apply

First run the intra-metric set round: for each metric set (M4, M7, M11), collect per-fold performance statistics (Accuracy and Pass/Fail F-measures) from the fivefold cross-validation (k=5, 10 repetitions) of the eight candidate algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, MANN, Random Forest trained with caret), apply Friedman's test, and if significant follow with Nemenyi's test using the scmamp package to pick the top algorithm per metric set. Then run the inter-metric set round: compare the three winning classifiers (e.g. AdaBoost/M4, Random Forest/M7, AdaBoost/M11) with the same Friedman-Nemenyi procedure on Pass and Fail F-measures to select the global classifier; in MetaClean this yielded AdaBoost with the M11 metric set (Pass p=0.001 and 0.005; Fail p=0.0023 and 0.0023). Collect the rank/statistics via getEvaluationMeasures() and visualize with getCDPlots() (critical-difference plots). Rationale: repeated measures across folds violate independence, so non-parametric Friedman with Nemenyi correction is appropriate for ranking many correlated classifiers as recommended by Demšar (2006).

## Related tools

- **scmamp** (R package used to run Friedman's test followed by Nemenyi's post-hoc test on the collected cross-validation performance statistics and to compute critical-difference plots)
- **caret** (Provides the eight classification algorithms whose cross-validated performance is ranked by the Friedman-Nemenyi procedure)
- **MetaClean** (Encapsulates the full pipeline including getEvaluationMeasures() and getCDPlots() that feed the statistical ranking rounds) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
R: ranks <- scmamp::friedmanTest(perf_table); scmamp::nemenyiTest(perf_table); MetaClean::getCDPlots(cv_results, metric = 'FMeasure')
```

## Evaluation signals

- Intra-metric set round: Friedman test reaches significance before Nemenyi correction is applied; Nemenyi p-values identify exactly one top algorithm per metric set (M4, M7, M11)
- Inter-metric set round: winner must be significantly better on both Pass and Fail F-measure (in MetaClean: p = 0.001, 0.005 for Pass and p = 0.0023, 0.0023 for Fail)
- Critical-difference plot (getCDPlots) should show non-overlapping mean-rank intervals between the selected global classifier and the runners-up
- Sanity check: rank ordering of the 24 classifiers should be consistent across the 10 repetitions × 5 folds (stable ranks, not driven by a single fold)
- Final selected classifier (metric set + algorithm) must be the one passed to trainClassifier() and match the reported best in the paper (AdaBoost with M11)

## Limitations

- The procedure ranks classifiers by aggregate CV statistics; in MetaClean it favored the metric set with the most Pass-class discriminative power, and the paper reported only moderately significant differences between metric pairs via Wilcoxon tests, so ranking may be sensitive to the chosen blocking factor
- Results depend on the development dataset (Metabolomics Workbench ST000726, 500 labeled peaks); classifier quality varied across independent test sets, with the Fail class F-measure suffering when low-quality peaks were scarce (e.g. Test 2's low-abundance peaks)
- Friedman-Nemenyi selection was validated on data from the same or similar LC–MS platform; performance on a different platform/matrix (as with Test 2's smaller peak count and IPO peak-width parameter) may not transfer
- Requires an XCMS object as pipeline input, so vendor-processed outputs cannot be fed into the upstream CV/evaluation steps that produce the ranking statistics

## Evidence
