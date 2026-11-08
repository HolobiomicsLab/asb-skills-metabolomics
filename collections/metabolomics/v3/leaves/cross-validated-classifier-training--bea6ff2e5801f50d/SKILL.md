---
name: cross-validated-classifier-training
description: Use when you have a manually labeled set of Pass/Fail peaks from an untargeted
  LC–MS metabolomics dataset and need to train, cross-validate, and statistically
  select among competing classifiers rather than trust a single model.
when_to_use_negative:
- Your data is vendor-processed output in a uniquely formatted file — MetaClean requires
  an XCMS object input and is not amenable to such formats in its current form.
- You have no labeled Pass/Fail peaks and cannot obtain expert annotations — fivefold
  cross-validation training of the classifier requires labeled examples.
- Your dataset contains many low-abundance Pass peaks resembling background noise
  but your training set lacks such examples — the classifier was shown to be conservative
  and performed worse on the low-quality class when trained without them.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaClean
  role: 'R package implementing the framework: runCrossValidation() for fivefold CV
    training, trainClassifier() and getPredicitons() for the final classifier, getEvaluationMeasures()
    and getCDPlots() for assessment'
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Upstream pre-processing (peak-picking, RT correction, grouping) that produces
    the XCMS object whose peaks are labeled and used for classifier training
- name: IPO
  role: Isotope-based XCMS parameter optimization applied to each dataset before labeling,
    ensuring consistent peak-picking inputs to classifier training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:a2fec517d7b599d1a556faa6c50f6b6b94733332c8265daf1c190ce41d99ee93
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# cross-validated-classifier-training

## Summary

Trains and selects a peak-quality classifier by pairing three sets of peak-quality metrics (Zhang et al. M4, TargetedMSQC-derived M7, and combined M11) with eight machine learning algorithms under a rigorous fivefold cross-validation framework, then statistically comparing candidates to pick the best classifier (here, AdaBoost on M11) for flagging low-quality peaks in untargeted LC–MS metabolomics.

## When to use

Use when you have a manually labeled set of Pass/Fail peaks from an untargeted LC–MS metabolomics dataset and need to train, cross-validate, and statistically select among competing classifiers rather than trust a single model.

## When NOT to use

- Your data is vendor-processed output in a uniquely formatted file — MetaClean requires an XCMS object input and is not amenable to such formats in its current form.
- You have no labeled Pass/Fail peaks and cannot obtain expert annotations — fivefold cross-validation training of the classifier requires labeled examples.
- Your dataset contains many low-abundance Pass peaks resembling background noise but your training set lacks such examples — the classifier was shown to be conservative and performed worse on the low-quality class when trained without them.

## Inputs

- XCMS-processed feature table (peak identification, retention time correction, and grouping already done) with pooled QC samples
- Expert-labeled Pass/Fail peak annotations for a development dataset (e.g. Metabolomics Workbench ST000726, ~500 labeled peaks)
- One or more independent labeled test datasets (e.g. ST000695, MTBLS354, MTBLS306 in positive and negative ionization modes)
- XCMS object (required by MetaClean for computing the 12 peak-quality metrics)

## Outputs

- A trained global peak-quality classifier (AdaBoost on the M11 metric set) from trainClassifier()
- Cross-validation performance statistics (Accuracy, Pass and Fail F-measures, sensitivity, specificity, PPV, NPV) for each of the 24 candidate classifiers
- Predicted Pass/Fail labels on independent test sets via getPredicitons()
- Friedman–Nemenyi significance rankings of candidate classifiers (e.g. Supplementary Fig. 3 scatter plots, CD plots via getCDPlots())

## How to apply

Expert-label ~500 peaks in an XCMS-processed feature table as Pass (high-quality) or Fail (low-quality), then generate candidate classifiers from three metric sets — Zhang et al. (M4), TargetedMSQC-derived (M7), and their combination (M11) — paired with eight diverse machine learning algorithms (24 candidates total). Train each candidate with a rigorous fivefold cross-validation (k=5 folds, repNum=10 repetitions via runCrossValidation() from the MetaClean R package), collecting F-measures for both Pass and Fail classes, sensitivity, specificity, PPV and NPV. Compare candidates in two statistical rounds — intra-metric set then inter-metric set — using Friedman's test followed by Nemenyi's post-hoc test on the collected performance statistics. The winning configuration in this paper was AdaBoost with the M11 metric set, which significantly outperformed alternatives (Pass class F-measure p = 0.001/0.005; Fail class p = 0.0023). Train the final global classifier on the full development set (e.g. 399 peaks from ST000726) via trainClassifier() and apply getPredicitons() to independent test sets, validating on same-platform (Test 1/ST000695) and cross-platform (MTBLS354, MTBLS306) datasets to confirm generalization. Finally, sanity-check against random labeling: randomly permute Pass/Fail labels 100 times, retrain, and confirm performance drops below the real model.

## Related tools

- **MetaClean** (R package implementing the framework: runCrossValidation() for fivefold CV training, trainClassifier() and getPredicitons() for the final classifier, getEvaluationMeasures() and getCDPlots() for assessment) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Upstream pre-processing (peak-picking, RT correction, grouping) that produces the XCMS object whose peaks are labeled and used for classifier training)
- **IPO** (Isotope-based XCMS parameter optimization applied to each dataset before labeling, ensuring consistent peak-picking inputs to classifier training)

## Examples

```
R: library(MetaClean); cv <- runCrossValidation(metrics, labels, k=5, repNum=10); best <- trainClassifier(train_metrics, train_labels); preds <- getPredicitons(best, test_metrics); getCDPlots(cv)
```

## Evaluation signals

- Fivefold CV of the final classifier must reproduce Friedman–Nemenyi significance for AdaBoost/M11 over other candidates (Pass F-measure p = 0.001/0.005; Fail F-measure p = 0.0023) across the 24 candidates.
- Predictions on Test 1 (ST000695, same platform) should show high Accuracy and Pass/Fail F-measures, consistent with the paper's qualitative report for that set.
- Training on randomly permuted Pass/Fail labels (100 permutations) must yield substantially lower F-measures than the real model — if not, the classifier is memorizing noise.
- Predictions on cross-platform test sets (MTBLS354, MTBLS306 positive/negative) should retain reasonable F-measures; a large drop for one set (as seen for Test 2) signals distribution shift that must be reported.
- Report both Pass and Fail F-measures — a model with high Pass but very low Fail F-measure indicates the conservative failure mode documented for Test 1 and 2.

## Limitations

- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality (Pass) one for Test 1 and Test 2, partly because the development dataset contained fewer low-quality peaks.
- Many misclassified low-quality peaks in Test 2 were low abundance and difficult to distinguish from background noise; the development dataset lacked many low-abundance Pass examples.
- MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs.
- Peak quality depends on the LC–MS run characteristics (ionization mode, analytical platform and method, biological matrix), so classifier performance may degrade when applied to datasets dissimilar to the development set.
- The paper reports performance qualitatively for some test sets (e.g. Test 1) and for the 24 candidate classifiers; exact metric values are not given in the text.

## Evidence
