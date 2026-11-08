---
name: lc-ms-pooled-qc-rsd-filtering
description: Use when an LC–MS feature table contains pooled quality-control (QC)
  sample injections and noisy, low-reproducibility peaks must be removed with a simple,
  unsupervised reproducibility filter before statistical analysis or classifier training.
when_to_use_negative:
- When integration-quality (not just reproducibility) must be judged — RSD filtering
  performed worse than the MetaClean global classifier and cannot flag badly integrated
  peaks that happen to be reproducible in QC.
- When the dataset has no routinely injected pooled QC samples, since RSD must be
  computed on pooled QCs.
- When vendor-processed outputs or non-XCMS feature tables without QC injections are
  all that is available.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaClean
  role: Provides the rsdFilter() function to compute pooled-QC RSD and apply the RSD
    < 30% threshold; also used to evaluate the filter via getEvaluationMeasures()
    and getCDPlots().
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Upstream peak-picking, retention time correction, and grouping that produces
    the feature table on which RSD is computed.
- name: IPO
  role: Optimizes XCMS parameters (e.g. peak width) before feature detection, which
    affects how many peaks pass the RSD threshold.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:d203ac6cb6a8e0f7c2489c3015d83d01bafb360558b85077397eb75222f7a09d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-ms-pooled-qc-rsd-filtering

## Summary

Filters untargeted LC–MS metabolomics peaks by computing the relative standard deviation (RSD) of each feature across routinely injected pooled QC samples and retaining only peaks below a pre-set threshold (typically RSD < 30%), removing noisy, poorly reproducible features before downstream statistics.

## When to use

Use when an LC–MS feature table contains pooled quality-control (QC) sample injections and noisy, low-reproducibility peaks must be removed with a simple, unsupervised reproducibility filter before statistical analysis or classifier training.

## When NOT to use

- When integration-quality (not just reproducibility) must be judged — RSD filtering performed worse than the MetaClean global classifier and cannot flag badly integrated peaks that happen to be reproducible in QC.
- When the dataset has no routinely injected pooled QC samples, since RSD must be computed on pooled QCs.
- When vendor-processed outputs or non-XCMS feature tables without QC injections are all that is available.

## Inputs

- Untargeted LC–MS peak/feature table (e.g. XCMS-processed) including pooled QC sample injections
- Pre-determined RSD threshold (typically 0.3 / 30%)
- Optional expert Pass/Fail peak-quality labels for evaluation

## Outputs

- Filtered feature table containing only peaks with pooled-QC RSD < 30%
- Per-peak RSD values and Pass/Fail (retain/filter) classification
- Evaluation measures (Accuracy, F-measures, sensitivity, specificity, PPV, NPV) if labels are available

## How to apply

Compute the RSD (relative standard deviation) of each peak's intensity across the pooled QC injections, treating the pooled QC as the reference for technical reproducibility. Apply the pre-determined threshold of RSD < 30% (0.3), as recommended by Dunn et al. and Want et al. and used as the standard baseline in this work, to retain reproducible peaks and flag the rest as Fail. On the MetaClean benchmark datasets this generated alternative Pass/Fail classifications for the 500 expert-labeled peaks per dataset and reduced them to RSD-passing subsets (e.g. 417, 75, 172, and 169 peaks in the test sets). Evaluate the filter against expert Pass/Fail labels using Accuracy, Pass/Fail F-measure, sensitivity, specificity, PPV, and NPV, and statistically rank it against alternative methods (e.g. the MetaClean global classifier) with Friedman–Nemenyi tests and CD plots. Rationale: RSD-on-QC is the most common, assumption-light filtering method, but it only captures variability and can mislabel peaks, since several peaks that passed RSD < 30% were still labeled Fail for integration quality.

## Related tools

- **MetaClean** (Provides the rsdFilter() function to compute pooled-QC RSD and apply the RSD < 30% threshold; also used to evaluate the filter via getEvaluationMeasures() and getCDPlots().) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Upstream peak-picking, retention time correction, and grouping that produces the feature table on which RSD is computed.)
- **IPO** (Optimizes XCMS parameters (e.g. peak width) before feature detection, which affects how many peaks pass the RSD threshold.)

## Examples

```
filtered <- rsdFilter(peakTable, rsdThreshold = 0.3)
```

## Evaluation signals

- Distribution of per-peak pooled-QC RSD values should show a clear subset below the 0.3 cutoff; peaks with RSD ≥ 0.3 must all be labeled Fail.
- Number of retained peaks should match expected RSD-passing counts (e.g. 417, 75, 172, 169 in the MetaClean test sets).
- Against expert Pass/Fail labels, evaluation measures (Accuracy, Pass/Fail F-measure, sensitivity, specificity, PPV, NPV) computed via getEvaluationMeasures() should reproduce the paper's RSD-filtering baseline.
- CD plots (getCDPlots()) should rank RSD filtering below the MetaClean global classifier in Pass/Fail F-measure, as in Fig. 4.
- Peaks passing RSD < 30% but labeled Fail by experts should be inspectable — their presence is expected and validates the filter's known blind spot.

## Limitations

- RSD filtering performed worse than the MetaClean global classifier on all test sets and cannot detect poor peak integrations.
- Several peaks that passed the RSD < 30% threshold were still labeled Fail for integration quality, so RSD alone leaves false positives.
- The filter is conservative and depends on pooled-QC injection quality; a smaller number of peaks passing RSD < 30% (as in Test 2) can signal dataset dissimilarity.
- Effectiveness depends on upstream XCMS parameter choices (e.g. IPO-optimized peak width) that alter feature detection before RSD is computed.

## Evidence
