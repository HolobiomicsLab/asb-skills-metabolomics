---
name: machine-learning-model-evaluation
description: Use when you have a labeled peak-quality development set and multiple
  metric-set × ML-algorithm candidate classifiers, and must choose one global classifier
  with statistically defensible evidence before deploying it on independent datasets.
when_to_use_negative:
- Input is a preprocessed feature table without XCMS objects or EIC points, so getEvalObj()
  cannot extract eicPts/eicPeakData/eicNos needed for metric computation.
- The dataset is vendor-processed output rather than an XCMS object — MetaClean requires
  XCMS object input and is not amenable to uniquely formatted vendor outputs.
- The goal is only reproducibility filtering on pooled QC peaks with a known threshold
  (e.g. RSD < 30%); the ML classifier evaluation is unnecessary if a simpler deterministic
  filter suffices and has been pre-validated.
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: caret
  role: Trains the eight candidate classification algorithms (Decision Tree, Logistic
    Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, MANN, Random Forest)
    under fivefold cross-validation
- name: scmamp
  role: Applies Friedman's test followed by Nemenyi's test to the collected performance
    statistics in two rounds to select the best classifier
- name: XCMS
  role: Produces the XCMS object containing EIC points and peak data that feed getEvalObj()
    and downstream metric computation
- name: IPO
  role: Optimizes XCMS peak-picking parameters on the datasets so that peak quality
    is comparable across the development and test sets
- name: TargetedMSQC
  role: Source of peak-quality metrics repurposed for untargeted metabolomics to form
    one of the candidate metric sets (M4/M7/M11)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:27e2d45a16e293f09bb18b59ce73ec4b3d8f34b92bba268218599787f120ecfa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - TargetedMSQC
  - XCMS
  - caret
  - scmamp
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# machine-learning-model-evaluation

## Summary

MetaClean's evaluation skill: train candidate classifiers under fivefold cross-validation, compare their Pass/Fail F-measures with Friedman/Nemenyi statistical tests in two rounds (intra-metric-set then inter-metric-set), and validate the selected classifier on independent held-out test sets rather than on the training splits alone.

## When to use

Use when you have a labeled peak-quality development set and multiple metric-set × ML-algorithm candidate classifiers, and must choose one global classifier with statistically defensible evidence before deploying it on independent datasets.

## When NOT to use

- Input is a preprocessed feature table without XCMS objects or EIC points, so getEvalObj() cannot extract eicPts/eicPeakData/eicNos needed for metric computation.
- The dataset is vendor-processed output rather than an XCMS object — MetaClean requires XCMS object input and is not amenable to uniquely formatted vendor outputs.
- The goal is only reproducibility filtering on pooled QC peaks with a known threshold (e.g. RSD < 30%); the ML classifier evaluation is unnecessary if a simpler deterministic filter suffices and has been pre-validated.

## Inputs

- Labeled development dataset (expert Pass/Fail annotations of ~500 peaks, e.g. Metabolomics Workbench ST000726)
- Peak-quality metric matrix (500×M with M4/M7/M11 metric subsets)
- Independent test datasets (Metabolomics Workbench ST000695, MetaboLights MTBLS354, MTBLS306 split by ionization mode)
- XCMS object with eicPts, eicPeakData, and eicNos from getEvalObj()

## Outputs

- Cross-validation results table (24 classifiers × metrics with accuracy and Pass/Fail F-measures)
- Friedman/Nemenyi statistical comparison results (two rounds: intra- and inter-metric-set)
- Trained global classifier object (AdaBoost with M11 metric set)
- Independent test-set evaluation table (accuracy plus Pass and Fail F-measures per test set)
- metaclean_pipeline_outputs.RDS containing evalObj, metric table, CV results, trained model, and test-set evaluation

## How to apply

Assemble a labeled development dataset (Metabolomics Workbench ST000726, 500 peaks labeled Pass/Fail by an expert) and compute peak-quality metrics to build the 500×M feature matrix. Train 24 candidate classifiers (combinations of the M4, M7, and M11 metric sets with eight algorithms from caret: Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, MANN, and Random Forest) using fivefold cross-validation with 10 repetitions. Collect accuracy and Pass/Fail F-measures, then run two rounds of Friedman's test followed by Nemenyi's test (scmamp package): first within each metric set, then across metric sets, to select the global classifier — AdaBoost with the M11 metric set won with F-measure p-values of 0.001–0.005 (Pass) and 0.0023 (Fail). Finally, train the selected classifier on the full development set and evaluate it on independent test sets (ST000695, MTBLS354, MTBLS306 split by ionization mode), reporting accuracy plus Pass and Fail F-measures; optionally also compare against the simpler RSD < 30% QC-filtering baseline.

## Related tools

- **caret** (Trains the eight candidate classification algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, SVM_Linear, AdaBoost, MANN, Random Forest) under fivefold cross-validation)
- **scmamp** (Applies Friedman's test followed by Nemenyi's test to the collected performance statistics in two rounds to select the best classifier)
- **XCMS** (Produces the XCMS object containing EIC points and peak data that feed getEvalObj() and downstream metric computation)
- **IPO** (Optimizes XCMS peak-picking parameters on the datasets so that peak quality is comparable across the development and test sets)
- **TargetedMSQC** (Source of peak-quality metrics repurposed for untargeted metabolomics to form one of the candidate metric sets (M4/M7/M11))

## Examples

```
runCrossValidation(evalObj, metricSet = "M11", k = 5, repNum = 10); getEvaluationMeasures(cvResults); getCDPlots(cvResults, test = "friedman")
```

## Evaluation signals

- Confirm 24 candidate classifiers were produced (3 metric sets × 8 algorithms) with fivefold CV and 10 repetitions before comparison.
- Verify two rounds of Friedman/Nemenyi tests were run: first intra-metric-set, then inter-metric-set, yielding a single global classifier.
- Check that the global classifier's Pass F-measure and Fail F-measure p-values are reported (expected: 0.001–0.005 for Pass, 0.0023 for Fail vs. runner-up classifiers).
- Confirm the final evaluation reports accuracy plus separate Pass and Fail F-measures on each of the four independent test sets, not just on CV splits.
- Optionally cross-check that the RSD < 30% baseline was computed on the same test sets for direct comparison against the ML classifier.

## Limitations

- The selected classifier is conservative for the low-quality (Fail) class compared to the high-quality (Pass) class on Test 1 and Test 2, partly because the development set contained fewer low-quality examples.
- Many misclassified low-quality peaks in Test 2 were low-abundance and hard to distinguish from background noise, since the development dataset lacked many low-abundance Pass peak examples.
- Test 2's dissimilarity (fewer peaks passing RSD < 30%, smaller IPO peak width parameter) suggests cross-dataset transfer of classifier performance is sensitive to analytical-platform and method differences.
- No exact performance metric values are reported in the article for the 24 candidate classifiers or for Test 1 (accuracy and F-measures are described qualitatively), limiting reproducibility of intermediate results.
- MetaClean requires an XCMS object input; vendor-processed outputs in other formats cannot be evaluated directly.

## Evidence
