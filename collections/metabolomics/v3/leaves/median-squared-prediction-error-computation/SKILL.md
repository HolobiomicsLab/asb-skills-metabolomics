---
name: median-squared-prediction-error-computation
description: Use when you need an outlier-sensitivity benchmark for normalization
  methods on simulated single-feature drift data where the true drift is known, e.g.
  comparing QC-RLSC, QC-RSC, rLOESS, rGAM, and tGAM across 0, 1, 2, or 4 injected
  outliers.
when_to_use_negative:
- Input is experimental data without a known ground-truth drift curve — MSPE against
  an expected pattern requires simulated data where the true drift is defined; use
  replicate concordance or drift/batch-effect inspection instead.
- The goal is to assess downstream differential abundance impact or QC-RSD filtering
  — those are separate tasks, not drift-prediction error benchmarks.
- Outlier positions or magnitudes cannot be randomized per run (e.g. single fixed
  scenario) — the median-over-200-runs design is defined for the 0/1/2/4-outlier grid
  with feature-specific outlier parameters.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Metanorm
  role: Provides the five normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM)
    whose drift predictions at QC indices feed the MSPE computation
  repo: https://github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: R implementation used to normalize each simulated feature under the benchmark
    grid; supports pleasingly parallel high-throughput normalization
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Environment in which the simulation (drift generation, outlier injection,
    200 runs), normalization calls, and MSPE aggregation are scripted
  repo: https://github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:4d07e25e2c052c1b4d47859c8200865b4e3cb815617738ddae20dd4f0814e5d7
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
  - Metanorm R package
  - R
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# median-squared-prediction-error-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark normalization methods on simulated data with known ground-truth drift by computing, at each QC index, the squared difference between a method's predicted and the expected drift pattern, then taking the median of these squared errors across 200 simulation runs (MSPE) per method, QC density, and outlier count.

## When to use

Use when you need an outlier-sensitivity benchmark for normalization methods on simulated single-feature drift data where the true drift is known, e.g. comparing QC-RLSC, QC-RSC, rLOESS, rGAM, and tGAM across 0, 1, 2, or 4 injected outliers.

## When NOT to use

- Input is experimental data without a known ground-truth drift curve — MSPE against an expected pattern requires simulated data where the true drift is defined; use replicate concordance or drift/batch-effect inspection instead.
- The goal is to assess downstream differential abundance impact or QC-RSD filtering — those are separate tasks, not drift-prediction error benchmarks.
- Outlier positions or magnitudes cannot be randomized per run (e.g. single fixed scenario) — the median-over-200-runs design is defined for the 0/1/2/4-outlier grid with feature-specific outlier parameters.

## Inputs

- Simulated single-feature intensity series (1000 samples, drift = 20 + sin(x) + N(0,0.3^2)) with known expected drift pattern
- QC-sample index vector (84 QCs = two per 10 biological samples, reducible to one per 10 for the lower density)
- Outlier injection specification: counts {0, 1, 2, 4}, positions (excluding first/last five observations), and magnitudes (subtract 20-30%, subtract 20-100%, or subtract 100% / add 200%)
- Normalized drift predictions at QC indices from each normalization method (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM)

## Outputs

- mspe_estimates.csv: table of 40 median squared prediction error rows (5 methods x 2 QC densities x 4 outlier counts)
- Per-method MSPE trajectories across outlier counts for comparison against the paper's simulation results (Figures S2-S4)

## How to apply

Simulate drift for 1000 samples per run as intensity_i = 20 + sin(x_i) + epsilon_i with epsilon ~ N(0, 0.3^2) at x_i = 0/999, 5/999, ..., 4995/999, and draw 84 QC intensities (two QC per 10 biological samples; optionally subset to one per 10 for a second QC density), offsetting the sine by 20 to avoid negative values; split the series into two equal parts for two distinct drift patterns. For each of k = 200 runs, inject 0, 1, 2, or 4 outliers at random locations (excluding the first and last five observations) with feature-specific magnitudes drawn as subtracting 20-30%, subtracting 20-100%, or subtracting 100% / adding 200% of the simulated intensity. Normalize each feature with all candidate Metanorm methods ('QC-RLSC', 'QC-RSC', 'rLOESS', 'rGAM', 'tGAM') under both QC densities. At all QC indices, compute the squared difference between each method's prediction and the expected drift pattern, take the median across the 200 runs, and assemble the 5 x 2 x 4 = 40 MSPE estimates into mspe_estimates.csv. The median (rather than mean) squared error is the point of the metric: it suppresses the influence of the very outlier events whose damage to non-robust methods is being quantified, giving a stable estimate of typical prediction error.

## Related tools

- **Metanorm** (Provides the five normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) whose drift predictions at QC indices feed the MSPE computation) — https://github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (R implementation used to normalize each simulated feature under the benchmark grid; supports pleasingly parallel high-throughput normalization) — https://github.com/UGent-LIMET/Metanorm
- **R** (Environment in which the simulation (drift generation, outlier injection, 200 runs), normalization calls, and MSPE aggregation are scripted) — https://github.com/UGent-LIMET/Metanorm

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- mspe_estimates.csv contains exactly 40 MSPE rows covering all method-by-QC-density-by-outlier-count combinations
- MSPE is computed only at QC indices and is the median across the 200 simulation runs, not a mean
- Robust methods (rLOESS, rGAM, tGAM) show stable, comparatively unaffected MSPE as the number of outliers increases from 0 to 4
- Non-robust QC-based methods (QC-RLSC, QC-RSC) show degrading MSPE as injected outliers accumulate
- Results are consistent with the paper's reported simulation results (Figures S2-S4)

## Limitations

- The simulation design does not yet mimic experimental metabolomics data (no resampling strategies or cross-metabolite correlations), as flagged by the authors as future work
- Benchmark neutrality must be maintained: care should be taken not to favor any of the benchmarked methods
- Outlier magnitudes/counts are limited to the specified grid (0/1/2/4; -20-30%, -20-100%, -100%/+200%), so conclusions may not extend to other outlier regimes

## Evidence
