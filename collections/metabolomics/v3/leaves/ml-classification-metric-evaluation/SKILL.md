---
name: ml-classification-metric-evaluation
description: Use when you have an XCMS-derived feature table of extracted ion chromatogram
  peaks with expert Pass/Fail quality labels and need to select and benchmark a machine
  learning classifier that flags low-quality peak integrations.
when_to_use_negative:
- Input is a vendor-processed output (e.g. MassHunter Profinder, Progenesis QI, Compound
  Discoverer) that is not an XCMS object — MetaClean requires an XCMS object input
  and cannot consume uniquely formatted vendor outputs in its current form.
- The dataset contains many low-abundance Pass peaks but few labeled examples of that
  class — the development dataset did not contain many such examples, and the classifier
  is conservative and performs worse for the low-quality (Fail) class than the high-quality
  one on Tests 1 and 2.
- The task is peak detection or alignment parameter optimization rather than quality
  classification of already-detected peaks — IPO or warpgroup are the appropriate
  tools for pre-filtering peak integration quality before classification.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: MetaClean
  role: R package implementing the peak-quality metric computation, classifier training,
    and Pass/Fail prediction for untargeted LC–MS peaks; provides the pre-trained
    AdaBoost+M11 global classifier
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Pre-processing raw LC–MS data (peak-picking, retention time correction, grouping)
    to produce the peak intensity matrix used as classifier input
  repo: https://github.com/sneumann/xcms
- name: IPO
  role: Optimizes XCMS parameters using isotope data to produce consistent peak-picking/retention
    correction before metric computation
- name: caret
  role: Unified interface for training the eight candidate classification algorithms
    with cross-validation and computing evaluation measures
- name: fastAdaboost
  role: Implements the AdaBoost algorithm paired with the winning M11 metric set
- name: ranger
  role: Implements the Random Forest algorithm used in the candidate classifier comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:874450d431c6cf3964836c5cd1b0c4a664d74f7b8958ba234bfb65405635a5f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  - caret
  - fastAdaboost
  - ranger
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ml-classification-metric-evaluation

## Summary

Evaluates supervised peak-quality classifiers for untargeted LC–MS metabolomics by pairing three peak-quality metric sets (Zhang M4, TargetedMSQC-derived M7, and combined M11) with eight ML algorithms, using fivefold cross-validation on the expert-labeled development dataset (MW id ST000726) and selecting the best classifier via Friedman/Nemenyi statistical tests on Accuracy, F-measure, Pass/Fail F-measure and AUC.

## When to use

Use when you have an XCMS-derived feature table of extracted ion chromatogram peaks with expert Pass/Fail quality labels and need to select and benchmark a machine learning classifier that flags low-quality peak integrations.

## When NOT to use

- Input is a vendor-processed output (e.g. MassHunter Profinder, Progenesis QI, Compound Discoverer) that is not an XCMS object — MetaClean requires an XCMS object input and cannot consume uniquely formatted vendor outputs in its current form.
- The dataset contains many low-abundance Pass peaks but few labeled examples of that class — the development dataset did not contain many such examples, and the classifier is conservative and performs worse for the low-quality (Fail) class than the high-quality one on Tests 1 and 2.
- The task is peak detection or alignment parameter optimization rather than quality classification of already-detected peaks — IPO or warpgroup are the appropriate tools for pre-filtering peak integration quality before classification.

## Inputs

- XCMS feature table (peaks × samples matrix) from the development dataset (Metabolomics Workbench ST000726) with expert Pass/Fail labels per peak (labeled by Petrick)
- XCMS feature table from independent test datasets: ST000695 (Test 1, same platform), MTBLS354 (Test 2), MTBLS306 positive mode (Test 3), MTBLS306 negative mode (Test 4)
- Peak-quality metric matrices (M4, M7, M11) computed per peak across samples and pooled QCs
- RSD < 30% filter on pooled QC samples as a comparative baseline (revised datasets containing only peaks with RSD < 30%)

## Outputs

- Trained global peak quality classifier (AdaBoost + M11) packaged in the MetaClean R package
- Cross-validation performance statistics per classifier (Accuracy, F-measure Pass, F-measure Fail, AUC) over 100 fivefold CV runs
- Friedman/Nemenyi statistical test results (p-values) ranking candidate classifiers within and across metric sets
- Test-set performance metrics (Accuracy, Pass/Fail F-measure) for the selected classifier on Test 1–4, benchmarked against RSD < 30% filtering

## How to apply

First compute the three metric sets on the labeled peaks: M4 from Zhang et al. (2014) — e.g. peak shape, signal-to-noise, and apex-related metrics — M7 from TargetedMSQC-derived metrics repurposed for untargeted metabolomics, and their union M11 (the MetaClean package's 12-peak-quality-metric set). For each metric set, train eight diverse algorithms (AdaBoost, Random Forest, AdaBoostM1, k-NN, logistic regression, decision tree, Naive Bayes, SVM) using the caret package, with fivefold cross-validation repeated over 100 partitions of the development dataset (ST000726). Record Accuracy, F-measure for the Pass class, F-measure for the Fail class, and AUC for each of the 24 metric-set × algorithm combinations, then run an intra-metric-set round of statistical comparison followed by an inter-metric-set round using Friedman's test with Nemenyi's post-hoc test (Demsar 2006) to identify the single global classifier. AdaBoost paired with M11 was selected (Friedman-Nemenyi p = 0.001–0.005 for Pass F-measure and 0.0023 for Fail F-measure). Evaluate the final global classifier on independent test sets — Test 1 (ST000695, same platform) and Tests 2–4 (MTBLS354, MTBLS306 pos/neg, different platforms) — reporting the same evaluation measures, and compare against the RSD < 30% filtering baseline on pooled QC samples. Validate that the selected classifier generalizes: Accuracy on cross-platform tests ranged from 0.65 (Test 2) to 0.79 (Test 4) and 0.75 (Test 3).

## Related tools

- **MetaClean** (R package implementing the peak-quality metric computation, classifier training, and Pass/Fail prediction for untargeted LC–MS peaks; provides the pre-trained AdaBoost+M11 global classifier) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Pre-processing raw LC–MS data (peak-picking, retention time correction, grouping) to produce the peak intensity matrix used as classifier input) — https://github.com/sneumann/xcms
- **IPO** (Optimizes XCMS parameters using isotope data to produce consistent peak-picking/retention correction before metric computation)
- **caret** (Unified interface for training the eight candidate classification algorithms with cross-validation and computing evaluation measures)
- **fastAdaboost** (Implements the AdaBoost algorithm paired with the winning M11 metric set)
- **ranger** (Implements the Random Forest algorithm used in the candidate classifier comparison)

## Examples

```
R> library(MetaClean)
R> metrics <- getM11Metrics(xcmsObj = xcms_object, label_list = c("Pass", "Fail"))
R> classifier <- trainMetaCleanClassifier(metricMat = metrics, labelVec = labels, classifier = "fastAdaboost")
```

## Evaluation signals

- For the intra- and inter-metric-set comparison, verify that Friedman/Nemenyi p-values are reported and that the winning classifier (AdaBoost+M11) is selected consistently across 100 fivefold CV partitions.
- For the global classifier's cross-platform generalization, confirm computed Accuracy on Test 2–4 is within ~0.05 of reported values (Test 2 = 0.65, Test 3 = 0.75, Test 4 = 0.79).
- Check that Pass and Fail F-measures are both reported for each test set and that the classifier's Fail F-measure is not drastically lower than its Pass F-measure for Tests 1–2 (conservative behavior expected).
- Compare the selected classifier's performance against the RSD < 30% filtering baseline on the same revised datasets and verify MetaClean outperforms or matches the threshold filter.
- Verify that the random-labeling permutation test (100 label permutations) produces performance substantially below the true classifier, confirming the model has learned genuine peak-quality signal.

## Limitations

- MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs.
- The classifier is conservative and performs worse for the low-quality (Fail) class than the high-quality one for Test 1 and 2, partly because the development dataset contained few examples of low-quality peaks.
- Many misclassified low-quality peaks in Test 2 were low-abundance and difficult to distinguish from background noise; the development dataset did not contain many low-abundance Pass peak examples.
- Peak quality depends on the LC–MS run context — ionization mode, analytical platform, and biological matrix — so the classifier may need re-evaluation or retraining when applied to runs with very different characteristics.
- Test 2 had a smaller number of peaks passing the RSD < 30% threshold and a smaller IPO peak width parameter, suggesting it was more dissimilar from the other datasets and reducing classifier performance there.

## Evidence

- [results] On cross-platform test sets, the AdaBoost+M11 classifier maintained high evaluation measures with Accuracy of 0.65 to 0.79, demonstrating it generalized reasonably well across analytical platforms.: [evidence span withheld — anchor status: not_found]
