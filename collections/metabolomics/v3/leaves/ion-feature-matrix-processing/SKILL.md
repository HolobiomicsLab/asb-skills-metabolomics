---
name: ion-feature-matrix-processing
description: Use when input is a pixel-by-ion feature matrix from an imaging mass
  spectrometry run (CSV with X/Y pixel coordinates plus per-pixel ion intensities)
  and noise ions must be removed during quality control before metabolite annotation
  or spatial pattern analysis.
when_to_use_negative:
- Input is raw instrument data (e.g. IMZML/MS runs) rather than a processed pixel-by-ion
  feature matrix in the SMAnalyst CSV layout
- The goal is annotation or differential analysis itself rather than the QC2 noise-ion
  filtering stage of the workflow
- Data are non-spatial bulk metabolomics measurements, since the quadrat test's CSR
  assessment requires spatial pixel coordinates
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Web/R platform whose QC2 module performs noise ion filtering with the adjustable
    noise-score threshold (default score ≥30); feature matrix upload is done via its
    Upload tab
  repo: mzlab-research/SManalyst
- name: spatstat
  role: R package (3.1-1) supplying the quadrat test used to score each ion's spatial
    distribution against Complete Spatial Randomness
- name: R
  role: Runtime (≥ v4.1.0) in which the feature matrix is loaded and per-ion quadrat
    tests are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:9697d89eca679623ff14a94b539c91973405db06fa9a2447f492228595a71917
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - SMAnalyst
  - spatstat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ion-feature-matrix-processing

## Summary

Processing step for spatial metabolomics feature matrices in which each ion's spatial distribution across pixels is tested for departure from Complete Spatial Randomness (spatstat quadrat test) and converted into a noise score (-log10 p-value), then filtered at SMAnalyst's default threshold of score ≥30 to separate noise ions from ions retained for downstream analysis.

## When to use

Use when input is a pixel-by-ion feature matrix from an imaging mass spectrometry run (CSV with X/Y pixel coordinates plus per-pixel ion intensities) and noise ions must be removed during quality control before metabolite annotation or spatial pattern analysis.

## When NOT to use

- Input is raw instrument data (e.g. IMZML/MS runs) rather than a processed pixel-by-ion feature matrix in the SMAnalyst CSV layout
- The goal is annotation or differential analysis itself rather than the QC2 noise-ion filtering stage of the workflow
- Data are non-spatial bulk metabolomics measurements, since the quadrat test's CSR assessment requires spatial pixel coordinates

## Inputs

- Feature matrix CSV for spatial metabolomics: columns 1-2 are X/Y pixel coordinates, remaining columns are ion intensities at each m/z value, one row per spatial pixel
- Noise-score threshold parameter (default: 30, adjustable in SMAnalyst)
- spatstat package (version 3.1-1) in R for the quadrat test

## Outputs

- noise_filter_results.csv with per-ion noise scores (-log10 of quadrat-test p-value) and noise/retained labels
- Noise-score distribution plot
- Filtered ion set retained for downstream annotation, spatial pattern discovery, and differential analysis

## How to apply

Load the feature matrix in R (columns 1-2 holding X/Y pixel coordinates, remaining columns per-pixel ion intensities; e.g. the mouse brain test dataset of 14,260 pixels × 3,044 ions). For each ion column, run the spatstat (3.1-1) quadrat test to assess whether the ion's spatial distribution conforms to Complete Spatial Randomness (CSR), reflecting the SMAnalyst QC2 noise-ion-filtering module. Convert each p-value to a noise score as -log10(p-value), then apply the default threshold of score ≥30: ions scoring below 30 are labeled as noise and removed, the rest are retained for downstream in-depth analysis. Write per-ion scores and labels to noise_filter_results.csv and plot the noise-score distribution; thresholds are adjustable in SMAnalyst but 30 is the default.

## Related tools

- **SMAnalyst** (Web/R platform whose QC2 module performs noise ion filtering with the adjustable noise-score threshold (default score ≥30); feature matrix upload is done via its Upload tab) — mzlab-research/SManalyst
- **spatstat** (R package (3.1-1) supplying the quadrat test used to score each ion's spatial distribution against Complete Spatial Randomness)
- **R** (Runtime (≥ v4.1.0) in which the feature matrix is loaded and per-ion quadrat tests are executed)

## Examples

```
Rscript -e 'r <- read.csv("feature_matrix.csv"); p <- sapply(r[,-(1:2)], function(i) quadrat.test(ppp(r[,1], r[,2], marks=NULL), i)$p.value); score <- -log10(p); keep <- score >= 30'
```

## Evaluation signals

- At the default threshold of 30, the removed/retained split should reproduce the paper's reported outcome: approximately 39% of noise ions filtered out and 61% of ions retained (Supplementary Figure S6)
- noise_filter_results.csv must contain one row per ion with a numeric noise score defined as -log10(quadrat-test p-value) and a binary noise/retained label
- Sum of removed and retained ion fractions must equal 1 across all 3,044 ion columns (for the mouse brain test dataset)
- Feature matrix parse check: row count equals pixel count (e.g. 14,260) and column count minus the 2 coordinate columns equals the ion count (e.g. 3,044)

## Limitations

- Ions below the default threshold of 30 are only labeled as potential noise ions; the default split retains the majority (61%) of noise ions, so threshold tuning is expected in practice
- The filtering depends on the CSR assumption from the spatstat quadrat test; ion scores and the removed/retained split are threshold-sensitive and tied to the tested dataset's spatial resolution (e.g. 100μm mouse brain section)
- Noise filtering is only one of four QC modules (QC1 background consistency, QC2 noise ion filtering, QC3 intensity distribution, QC4 missing values) and must be preceded by tissue/background region selection

## Evidence

- [readme] readme: "SManalyst requires a **feature matrix** in CSV format: *   **Columns 1-2**: Pixel coordinates (X, Y)"
- [intro] intro: "Adjust noise ion threshold (default: score ≥30)"
