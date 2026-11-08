---
name: peak-quality-metric-calculation
description: Use when an untargeted LC-MS peak table has been generated with XCMS
  (peak-picking, retention time correction, grouping across samples and pooled QCs)
  and you need quantitative, per-peak integration-quality features to train or apply
  the MetaClean peak-quality classifier.
when_to_use_negative:
- Your peak list comes from vendor-processed outputs (e.g. MassHunter Profinder, Progenesis
  QI, Compound Discoverer) rather than an XCMS object — MetaClean cannot consume uniquely
  formatted vendor outputs in its current form.
- You only need simple reproducibility-based noise removal rather than integration-quality
  assessment — a direct RSD < 30% cut-off in pooled QC samples may suffice without
  computing metric features.
- Your data have not yet been through XCMS peak-picking, retention time correction,
  and grouping — the metrics require the integrated EICs and grouped peak information
  an XCMS object provides.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MetaClean
  role: R package that computes the 12 peak-quality metrics from an XCMS object and
    pairs them with machine learning algorithms to build the peak-quality classifier
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Produces the pre-processed feature table (peak-picking, retention time correction,
    grouping across samples and QCs) that serves as the required input for metric
    calculation
- name: IPO
  role: Optimizes XCMS parameters (e.g. peak width) using isotope data before metric
    calculation, since integration quality depends on these parameters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:cdd4c917cf84da6f77c41fca895c3005e18cf100993885e37df53b93fd694014
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# peak-quality-metric-calculation

## Summary

Computes the MetaClean peak-quality metrics (a combination of Zhang et al. 2014 and TargetedMSQC-derived metrics) from an XCMS object to characterize chromatographic peak integration quality in untargeted LC-MS metabolomics data, as a prerequisite for machine-learning-based Pass/Fail peak classification.

## When to use

Use when an untargeted LC-MS peak table has been generated with XCMS (peak-picking, retention time correction, grouping across samples and pooled QCs) and you need quantitative, per-peak integration-quality features to train or apply the MetaClean peak-quality classifier.

## When NOT to use

- Your peak list comes from vendor-processed outputs (e.g. MassHunter Profinder, Progenesis QI, Compound Discoverer) rather than an XCMS object — MetaClean cannot consume uniquely formatted vendor outputs in its current form.
- You only need simple reproducibility-based noise removal rather than integration-quality assessment — a direct RSD < 30% cut-off in pooled QC samples may suffice without computing metric features.
- Your data have not yet been through XCMS peak-picking, retention time correction, and grouping — the metrics require the integrated EICs and grouped peak information an XCMS object provides.

## Inputs

- XCMS object (xcmsSet/xset) resulting from peak-picking, retention time correction, and grouping of peaks across samples and pooled quality-control (QC) injections
- Untargeted LC-MS metabolomics raw data pre-processed with XCMS (optionally with parameters optimized via IPO)

## Outputs

- Per-peak peak-quality metric values for the 12 MetaClean metrics (Zhang et al. 2014 and TargetedMSQC-derived metrics)
- Feature table suitable for input to the MetaClean classification framework (trainClassifier(), getPredicitons(), runCrossValidation())

## How to apply

Start from the XCMS-processed feature table produced by peak-picking, retention time correction, and grouping of peaks across samples and pooled QC injections (optionally with IPO-optimized XCMS parameters). For each peak, compute the 12 MetaClean peak-quality metrics: metrics adapted from Zhang et al. (2014) — including measures of peak shape, width, and intensity — and metrics from TargetedMSQC (Eshghi et al. 2018) repurposed for untargeted data. Three metric-set variants are supported in the framework: the Zhang-only set, the TargetedMSQC-derived set, and the combined set (M11), where the combined set gave the best-performing classifier (AdaBoost M11) under fivefold cross-validation. These metrics serve as the feature matrix input to trainClassifier() or getPredicitons(), and are exposed through the MetaClean R package. Because the metrics depend on integration boundaries and peak grouping quality, ensure the XCMS object is the direct output of the same pre-processing run you intend to filter — MetaClean requires an XCMS object input and cannot consume vendor-processed outputs such as MassHunter Profinder or Progenesis QI tables.

## Related tools

- **MetaClean** (R package that computes the 12 peak-quality metrics from an XCMS object and pairs them with machine learning algorithms to build the peak-quality classifier) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Produces the pre-processed feature table (peak-picking, retention time correction, grouping across samples and QCs) that serves as the required input for metric calculation)
- **IPO** (Optimizes XCMS parameters (e.g. peak width) using isotope data before metric calculation, since integration quality depends on these parameters)

## Examples

```
remotes::install_github("KelseyChetnik/MetaClean")
```

## Evaluation signals

- The computed metric table should contain one row per peak from the XCMS object and 12 metric columns covering both the Zhang et al. (2014) metrics and the TargetedMSQC-derived metrics.
- Expert-labeled Pass (high-quality) peaks should show systematically better (e.g. higher signal-to-noise, smoother shape, narrower m/z deviation) metric distributions than Fail (low-quality) peaks; CD plots built with getCDPlots() should show the metric sets separating Pass from Fail classes.
- Metrics must be computed on the exact same XCMS object used downstream for classification; re-running peak-picking or grouping after metric calculation will invalidate alignment between metric values and peak identities.
- If IPO-optimized XCMS parameters are used, the peak width parameter should match the value reported in the pre-processing run (e.g. the smaller IPO peak width parameter in Test 2 explains its dissimilarity).
- When the combined M11 metric set is used, cross-validated AdaBoost M11 should achieve significantly higher Pass and Fail F-measures than the Zhang-only or TargetedMSQC-only metric sets (Friedman-Nemenyi p = 0.001–0.005 as reported).

## Limitations

- MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs, limiting applicability to data pre-processed outside the XCMS ecosystem.
- Peak-quality characteristics depend on the ionization mode, analytical platform and method, and biological matrix; metrics computed on one dataset may transfer poorly to a dissimilar one (e.g. Test 2 had fewer peaks passing RSD < 30% and a smaller IPO peak width parameter).
- The development dataset contained few examples of low-abundance Pass peaks, making some metrics (particularly those sensitive to low abundance versus background noise) hard to calibrate on such peaks.
- Metric calculation is a prerequisite, not the end step — classifier predictions should still be treated as conservative for the low-quality (Fail) class, as the MetaClean global classifier performed worse for Fail than Pass on some test sets.

## Evidence
