---
name: untargeted-metabolomics-peak-classifier-inference
description: Use when you have an XCMS-processed untargeted LC–MS feature table with
  peak-quality metrics and want to automatically flag low-quality peak integrations
  without manual curation.
when_to_use_negative:
- Input is a feature table already manually curated or produced by vendor software
  (MassHunter Profinder, Progenesis QI, Compound Discoverer) — MetaClean requires
  an XCMS object input and cannot consume uniquely formatted vendor-processed outputs.
- The dataset was generated on a platform or in an ionization mode very different
  from the classifier's development data (e.g. Test 2-like conditions with few RSD-passing
  peaks) — cross-platform generalization may degrade, as observed for low-abundance
  Fail peaks.
- Goal is targeted metabolomics with a priori known analytes — the M11 metrics were
  repurposed from targeted QC specifically for untargeted LC–MS, not for confirming
  known compounds.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaClean
  role: R package that supplies the pre-trained classifiers (including AdaBoost with
    the M11 metric set) and functions to compute peak-quality metrics and flag low-quality
    peaks in XCMS pipelines
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Pre-processor that generates the peak-picked, RT-corrected, grouped feature
    table (xcmsSet object) that the classifier consumes
- name: IPO
  role: Optimizes XCMS peak-picking, retention time correction, and grouping parameters
    using isotope data before classification
- name: fastAdaboost
  role: Implements the AdaBoost algorithm paired with the M11 metric set for the best-performing
    global classifier
- name: caret
  role: Used to train and evaluate candidate classifiers within the cross-validation
    framework
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:61569dfdc4eeef358e1a02b4628fc3d7f14c26f33a73975cd344240a758dd681
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  - caret
  - fastAdaboost
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# untargeted-metabolomics-peak-classifier-inference

## Summary

Applies a pre-trained machine-learning peak-quality classifier (e.g. the MetaClean AdaBoost model built on the 11-metric M11 set) to an XCMS feature table to binarize every integrated chromatographic peak as Pass (high-quality) or Fail (low-quality), replacing manual inspection of peak integrations.

## When to use

Use when you have an XCMS-processed untargeted LC–MS feature table with peak-quality metrics and want to automatically flag low-quality peak integrations without manual curation.

## When NOT to use

- Input is a feature table already manually curated or produced by vendor software (MassHunter Profinder, Progenesis QI, Compound Discoverer) — MetaClean requires an XCMS object input and cannot consume uniquely formatted vendor-processed outputs.
- The dataset was generated on a platform or in an ionization mode very different from the classifier's development data (e.g. Test 2-like conditions with few RSD-passing peaks) — cross-platform generalization may degrade, as observed for low-abundance Fail peaks.
- Goal is targeted metabolomics with a priori known analytes — the M11 metrics were repurposed from targeted QC specifically for untargeted LC–MS, not for confirming known compounds.

## Inputs

- XCMS-processed LC–MS data (xcmsSet/XCMS object) from untargeted metabolomics runs
- Peak-quality metric matrix (e.g. the 11-metric M11 set: 7 TargetedMSQC-derived metrics + 4 Zhang et al. metrics)
- Pre-trained peak-quality classifier (e.g. AdaBoost with M11 metric set, shipped in the MetaClean R package)
- Optimized decision threshold from cross-validation for binarizing predictions

## Outputs

- Per-peak Pass/Fail (high-quality/low-quality) classification labels
- Filtered feature table with low-quality peaks removed
- Predicted probabilities per peak from the classifier
- Evaluation measures (Accuracy, Pass F-measure, Fail F-measure) against expert labels

## How to apply

Compute peak-quality metrics for every peak in the pre-processed dataset: here the M11 set combines seven TargetedMSQC-derived metrics (adapted from Eshghi et al. 2018) plus the four Zhang et al. (2014) metrics extracted from the XCMS peak list. Load the pre-trained global peak-quality classifier — in this paper AdaBoost paired with the M11 metric set — and apply it to the metric matrix, binarizing probabilistic predictions at the threshold that maximized balanced accuracy during fivefold cross-validation. The classifier labels each peak Pass (high-quality) or Fail (low-quality); peaks labeled Fail are removed from the feature table before downstream analyses. Optionally add a second automated filter layer, such as RSD < 30% computed on pooled QC samples. Correct application is judged by the classifier's F-measures on expert-labeled Pass and Fail classes (global classifier Accuracy 0.65–0.79 across cross-platform test sets) rather than by manual re-inspection of every integrated peak.

## Related tools

- **MetaClean** (R package that supplies the pre-trained classifiers (including AdaBoost with the M11 metric set) and functions to compute peak-quality metrics and flag low-quality peaks in XCMS pipelines) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Pre-processor that generates the peak-picked, RT-corrected, grouped feature table (xcmsSet object) that the classifier consumes)
- **IPO** (Optimizes XCMS peak-picking, retention time correction, and grouping parameters using isotope data before classification)
- **fastAdaboost** (Implements the AdaBoost algorithm paired with the M11 metric set for the best-performing global classifier)
- **caret** (Used to train and evaluate candidate classifiers within the cross-validation framework)

## Examples

```
remotes::install_github("KelseyChetnik/MetaClean"); pred <- predictQCCV(xcmsObj, model="AdaBoost_M11")
```

## Evaluation signals

- Per-peak labels are binary Pass/Fail and every peak in the XCMS feature table receives exactly one label.
- On cross-platform test sets, Accuracy of the global classifier should fall in the reported 0.65–0.79 range (Test 2 = 0.65, Test 3 = 0.75, Test 4 = 0.79) when evaluated against expert Pass/Fail labels.
- F-measure for the Fail (low-quality) class should be reported and compared with the Pass class; the classifier was conservative, with Fail F-measure lower than Pass F-measure on Test 1 and 2.
- Prediction probabilities, when thresholded, should use the CV-optimized threshold that maximized balanced accuracy, not a default 0.5 cutoff.
- Randomized-label sanity check: training on 100 label-permuted versions of the development data should produce markedly worse performance, confirming the classifier learned genuine peak-quality signal.

## Limitations

- The classifier requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs.
- Performance was worse for the low-quality (Fail) class than the high-quality (Pass) class on Test 1 and 2, partly because the development set contained few low-quality peak examples.
- Many misclassified low-quality peaks in Test 2 were low-abundance and difficult to distinguish from background noise; the development dataset did not contain many low-abundance Pass peak examples.
- Ionization mode, analytical platform, method, and biological matrix affect peak shape and integration quality, so cross-platform applicability is not guaranteed without assessment.
- The classifier can be combined with traditional RSD < 30% filtering on pooled QCs as an additional layer, but the two approaches target different quality aspects.

## Evidence

- [results] Maintained accuracy across cross-platform test sets: [evidence span withheld — anchor status: not_found]
- [readme] Metric set provenance in README: [evidence span withheld — anchor status: not_found]
