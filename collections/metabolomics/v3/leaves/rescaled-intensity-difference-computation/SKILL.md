---
name: rescaled-intensity-difference-computation
description: Use when benchmarking metabolomics normalization methods on replicate
  runs (especially between-batch replicates) and you need a robust concordance metric
  that avoids favoring any method, as an alternative to QC-sample RSD or differential-abundance-count
  benchmarks.
when_to_use_negative:
- When there are no replicate pairs (e.g., only QC samples and single biological samples)
  — the metric is defined on between-replicate intensity differences and cannot be
  computed.
- When intensities are on the raw (non-log) scale — rescaled differences on raw intensities
  are not comparable to the log-transformed, base e intensities used in the article's
  computation.
- As a sole benchmark when multivariate methods (SERFF, TIGER) or a broader set of
  normalizations are of interest but the replicate structure is absent — the article
  flags that such benchmarking requires additional strategies.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Metanorm
  role: R package used to produce the normalized intensity outputs (tGAM, rGAM, rLOESS,
    QC-RLSC, QC-RSC) whose rescaled differences and MADs are then computed; outputs
    can be loaded and processed in R
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Computational environment in which the rescaled differences and per-method
    MADs are calculated (using Metanorm-processed outputs)
  repo: https://github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:122e3b330b3b8ca612d9fa2f1d51843f63c6808a83649123d08431b249aebb5e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# rescaled-intensity-difference-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies between-replicate concordance after normalization by computing, per replicate pair and compound, the difference between the two observed intensities rescaled by their mean intensity, then aggregating these into a MAD per normalization approach (tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, QC-RSC, unnormalized). Lower MAD indicates better correspondence of intensities across batches; this is the metric that showed the robust methods reducing MAD by 49–52% versus unnormalized data on BioHEART between-batch replicate runs.

## When to use

Use when benchmarking metabolomics normalization methods on replicate runs (especially between-batch replicates) and you need a robust concordance metric that avoids favoring any method, as an alternative to QC-sample RSD or differential-abundance-count benchmarks.

## When NOT to use

- When there are no replicate pairs (e.g., only QC samples and single biological samples) — the metric is defined on between-replicate intensity differences and cannot be computed.
- When intensities are on the raw (non-log) scale — rescaled differences on raw intensities are not comparable to the log-transformed, base e intensities used in the article's computation.
- As a sole benchmark when multivariate methods (SERFF, TIGER) or a broader set of normalizations are of interest but the replicate structure is absent — the article flags that such benchmarking requires additional strategies.

## Inputs

- Replicate-run intensity matrix: rows = compounds, columns = sample runs, containing pairs of identical biological samples run in subsequent batches (BioHEART between-batch replicate runs)
- Unnormalized intensities (log-transformed, base e, prior to normalization)
- Normalized intensities from each benchmarked method: tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, QC-RSC (produced via Metanorm processing)
- Sample-type annotations distinguishing QC samples from biological samples

## Outputs

- replicate_concordance_mad.tsv — per-method table of MAD of rescaled differences and relative % reduction versus the unnormalized baseline

## How to apply

Load the replicate-run intensity matrix — pairs of identical biological samples run in subsequent batches — with intensities that are log-transformed (base e) prior to normalization, from each normalization approach being compared (tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, QC-RSC) plus the unnormalized intensities as baseline. For each replicate pair and compound, calculate the difference between the two observed intensities rescaled by their mean intensity (i.e., (I1 − I2) / mean(I1, I2)). Compute the MAD (median absolute deviation) of these rescaled differences per normalization approach, in R using Metanorm-processed outputs. Then compute each method's relative reduction in MAD versus the unnormalized baseline as a percentage. Tabulate per-method MAD and % reduction, and save as replicate_concordance_mad.tsv. The article explicitly proposes this as a benchmark alternative to common pitfalls such as using a reduction of QC-sample relative standard deviation or an increased number of differentially abundant metabolites as proxies for good normalization, and stresses that care should be taken to ensure neutrality — not potentially favoring any of the benchmarked methods.

## Related tools

- **Metanorm** (R package used to produce the normalized intensity outputs (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC) whose rescaled differences and MADs are then computed; outputs can be loaded and processed in R) — https://github.com/UGent-LIMET/Metanorm
- **R** (Computational environment in which the rescaled differences and per-method MADs are calculated (using Metanorm-processed outputs)) — https://github.com/UGent-LIMET/Metanorm

## Examples

```
load(system.file("extdata", "example.RData", package = "metanorm")); normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, batch = batch); # then compute per-pair rescaled differences (I1 − I2)/mean(I1, I2) and MAD(normdat_diffs) per method
```

## Evaluation signals

- Robust methods (rLOESS, rGAM, tGAM) should show a 49–52% reduction in MAD of rescaled differences versus unnormalized data.
- SERFF should show a 41% reduction; TIGER a 40% reduction.
- QC-RLSC and QC-RSC should show 18–22% reductions.
- Lower MAD corresponds to better correspondence of intensities across batches; verify the ranking matches the article's expectation that robust methods achieve the lowest between-replicate normalized differences.
- Confirm the output TSV (replicate_concordance_mad.tsv) contains one MAD and one % reduction per benchmarked method, with the unnormalized baseline reduction set to 0%.

## Limitations

- Care should be taken to ensure neutrality — not potentially favoring any of the benchmarked methods — so the metric must not be combined with method-specific tuning before comparison.
- Outliers (occasional, disproportionately high or low metabolite intensities) can deteriorate the performance of existing QC-sample-based normalization methods, which is precisely what this metric is designed to reveal; interpretation should account for method-specific outlier sensitivity.
- The article recommends inclusion of both QC and replicate samples to support data quality and normalization performance investigations; absence of either limits the interpretability of this metric.
- Multivariate methods (SERFF, TIGER) and a broader set of normalization methods remain as important future work for more comprehensive benchmarking.

## Evidence

- [results] rescaled-difference MAD definition: [evidence span withheld — anchor status: not_found]
- [results] expected reduction ranges: [evidence span withheld — anchor status: not_found]
- [results] robust methods best concordance: [evidence span withheld — anchor status: not_found]
- [readme] Metanorm provides normalized outputs in R: [evidence span withheld — anchor status: external_doc]
