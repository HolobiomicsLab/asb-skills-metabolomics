---
name: spatial-randomness-quadrat-testing
description: Use when a spatial metabolomics feature matrix (pixel X/Y coordinates
  + per-pixel ion intensities, e.g. 14,260 pixels x 3,044 ions from an AFAD-ESI(+)
  mouse brain IMS section) must be QC-filtered of noise ions before annotation or
  spatial pattern analysis.
when_to_use_negative:
- The dataset is already QC-filtered or the user's goal is metabolite annotation,
  clustering, or differential analysis rather than the QC2 noise-ion filtering step
- The input lacks spatial pixel coordinates (columns 1-2 X/Y), so no spatial point
  pattern exists for a quadrat/CSR test
- The user wants background-consistency evaluation (QC1), intensity distribution (QC3),
  or missing-value assessment (QC4) instead of noise-ion filtering
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SMAnalyst
  role: Web/R platform implementing the QC2 noise-ion filtering step whose default
    threshold is noise score >= 30
  repo: https://github.com/mzlab-research/SManalyst
- name: spatstat
  role: R package whose quadrat test assesses each ion's spatial distribution against
    Complete Spatial Randomness to derive the noise score
- name: R
  role: Runtime environment (>= v4.1.0) in which the per-ion quadrat tests and -log10
    scoring are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:4521443e09d7b60b5db10c3eb021b2e978dcd0a4f6330e2b81dbe18a02c7cfec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - SMAnalyst
  - SMQVP
  - spatstat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spatial-randomness-quadrat-testing

## Summary

Noise-ion QC step from SMAnalyst that scores each ion's spatial distribution against Complete Spatial Randomness (CSR) using the spatstat quadrat test, converts p-values to -log10 noise scores, and filters ions with scores below the default threshold of 30.

## When to use

Use when a spatial metabolomics feature matrix (pixel X/Y coordinates + per-pixel ion intensities, e.g. 14,260 pixels x 3,044 ions from an AFAD-ESI(+) mouse brain IMS section) must be QC-filtered of noise ions before annotation or spatial pattern analysis.

## When NOT to use

- The dataset is already QC-filtered or the user's goal is metabolite annotation, clustering, or differential analysis rather than the QC2 noise-ion filtering step
- The input lacks spatial pixel coordinates (columns 1-2 X/Y), so no spatial point pattern exists for a quadrat/CSR test
- The user wants background-consistency evaluation (QC1), intensity distribution (QC3), or missing-value assessment (QC4) instead of noise-ion filtering

## Inputs

- Feature matrix CSV from imaging mass spectrometry: columns 1-2 are pixel X/Y coordinates, subsequent columns are per-pixel ion intensities at each m/z value (one row per spatial pixel)
- Example scale: mouse brain coronal section, AFAD-ESI (+) mode, 100 um resolution, 14,260 pixels x 3,044 ions (NGDC OMIX OMIX009541)

## Outputs

- Per-ion noise scores and retained/removed labels written to noise_filter_results.csv
- Filtered feature matrix retaining only ions with noise score >= 30 for downstream analysis
- Plot of the noise-score distribution (cf. Supplementary Figure S6)

## How to apply

Load the feature matrix in R with columns 1-2 as X/Y pixel coordinates and remaining columns as per-pixel ion intensities. For each ion column, run the spatstat (v3.1-1) quadrat test to test whether the ion's spatial pixel distribution deviates from Complete Spatial Randomness (CSR); ions indistinguishable from CSR are candidate noise. Convert each test's p-value to a noise score as the negative logarithm base 10 (-log10 p). Apply the SMAnalyst default threshold score >= 30: ions scoring below the threshold are labeled noise and removed, while ions at or above 30 are retained for downstream in-depth analysis. Summarize the removed/retained fractions, write per-ion scores and labels to noise_filter_results.csv, and plot the noise-score distribution (the threshold is adjustable in the SMAnalyst QC2 'Noise ion filtering' step).

## Related tools

- **SMAnalyst** (Web/R platform implementing the QC2 noise-ion filtering step whose default threshold is noise score >= 30) — https://github.com/mzlab-research/SManalyst
- **spatstat** (R package whose quadrat test assesses each ion's spatial distribution against Complete Spatial Randomness to derive the noise score)
- **R** (Runtime environment (>= v4.1.0) in which the per-ion quadrat tests and -log10 scoring are executed)

## Examples

```
library(spatstat); pv <- quadrat.test(ppp(df$X, df$Y, owin(xrange, yrange)), nsim=199)$p.value; score <- -log10(pv)
```

## Evaluation signals

- At the default threshold of 30 on Dataset 1, the filtered/retained split reproduces the reported ~39% of noise ions filtered out and ~61% retained (Supplementary Figure S6)
- Each noise score equals the negative logarithm base 10 of the ion's quadrat-test p-value; scores >= 30 correspond to very small p-values and must be retained
- noise_filter_results.csv exists, contains one score + label per input ion, and the label counts sum to the total ion count (3,044 for the example dataset)
- The noise-score distribution plot spans the observed score range and shows the threshold-30 cut position
- All 3,044 ion columns were tested with spatstat quadrat.test (v3.1-1) with no untested ion columns remaining

## Limitations

- The default threshold of 30 is deliberately conservative: on Dataset 1, 61% of noise ions are still retained for downstream analysis
- CSR-based noise scoring assumes spatial randomness is the null for noise ions; biologically diffuse (spatially homogeneous) true ions may be misclassified as noise
- Performance and defaults were validated on a single mouse brain AFAD-ESI (+) 100 um dataset (14,260 pixels x 3,044 ions); other acquisition modes/resolutions may need threshold adjustment
- The article provides no changelog or discussion-section caveats beyond the QC workflow itself

## Evidence

- [intro] default noise threshold: "Adjust noise ion threshold (default: score ≥30)"
- [readme] QC2 noise filtering: "Noise ion filtering (QC2)"
- [readme] spatstat dependency: ""shinyBS", "sf", "spdep", "spatstat", "MetaboCoreUtils", "Seurat""
- [readme] example dataset dimensions: "Content: 14,260 pixels × 3,044 ions"
- [abstract] scale handling claim: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
