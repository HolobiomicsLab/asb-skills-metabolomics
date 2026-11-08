---
name: outlier-injection-benchmarking
description: Use when you must quantify whether outlier counts or magnitudes degrade
  QC-based normalization methods (e.g., QC-RLSC, QC-RSC) relative to robust alternatives
  (rLOESS, rGAM, tGAM) on data with a known drift pattern.
when_to_use_negative:
- You only need to normalize real experimental data and suspect outliers — use the
  robust methods directly (e.g., tGAM via metanorm()) rather than running the full
  injection benchmark
- Your normalization evaluation uses QC-sample RSD reduction or differential-abundance
  counts as proxies — the article flags these as common benchmarking pitfalls to avoid
- You lack a known ground-truth drift pattern; MSPE against expected drift is meaningless
  on data without simulated ground truth
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: Metanorm
  role: R package implementing the five normalization methods (QC-RLSC, QC-RSC, rLOESS,
    rGAM, tGAM) applied to each simulated, outlier-contaminated feature
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Language in which the drift simulation, outlier injection, 200-run loop, and
    MSPE computation are executed (R >= 4.4.0 required)
  repo: https://github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Installed via pak::pak("UGent-LIMET/Metanorm") to call metanorm() with model
    = 'QC-RLSC' | 'QC-RSC' | 'rLOESS' | 'rGAM' | 'tGAM' on the simulated matrix
  repo: https://github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:1e537389adc3d7a804f207421983e2eab44f4202cfbf66ff93148a46ae0eef74
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# outlier-injection-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark metabolomics normalization methods under controlled outlier contamination by simulating drift with known ground truth, injecting 0/1/2/4 outliers at random positions, and comparing methods via median squared prediction error (MSPE) at QC indices using the Metanorm R package.

## When to use

Use when you must quantify whether outlier counts or magnitudes degrade QC-based normalization methods (e.g., QC-RLSC, QC-RSC) relative to robust alternatives (rLOESS, rGAM, tGAM) on data with a known drift pattern.

## When NOT to use

- You only need to normalize real experimental data and suspect outliers — use the robust methods directly (e.g., tGAM via metanorm()) rather than running the full injection benchmark
- Your normalization evaluation uses QC-sample RSD reduction or differential-abundance counts as proxies — the article flags these as common benchmarking pitfalls to avoid
- You lack a known ground-truth drift pattern; MSPE against expected drift is meaningless on data without simulated ground truth

## Inputs

- In silico simulated drift series: 1000 samples per run, intensity_i = 20 + sin(x_i) + epsilon_i, epsilon ~ N(0, 0.3^2)
- 84 QC intensities drawn from a sine function offset by 20, two QC samples per 10 biological samples
- Outlier injection specifications: counts (0, 1, 2, 4) and magnitude rules (subtract 20-30%, subtract 20-100%, subtract 100% / add 200%)
- Metanorm method list: QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM

## Outputs

- mspe_estimates.csv with exactly 40 rows (5 methods x 2 QC densities x 4 outlier counts) of median squared prediction errors
- Method-by-QC-density-by-outlier-count comparison table mirroring the paper's simulation results (Figures S2-S4)

## How to apply

Simulate drift for 1000 samples per run as intensity_i = 20 + sin(x_i) + epsilon_i with epsilon ~ N(0, 0.3^2), and draw 84 QC intensities (two per 10 biological samples) from an offset sine function, splitting the series into two parts for two drift patterns. For each of k = 200 runs, inject zero, one, two, or four outliers at random locations (excluding the first and last five observations), with magnitudes set by subtracting 20-30%, subtracting 20-100%, or subtracting 100% / adding 200% of the simulated intensity, drawing outlier parameters feature-specifically per run. Normalize each simulated feature with the five Metanorm methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) using both one and two QC samples per 10 biological samples. At each QC index compute the squared difference between the method's prediction and the expected drift, then take the median across the 200 runs to obtain MSPE per method; assemble the 5 methods x 2 QC densities x 4 outlier counts = 40 MSPE values. The rationale is that only in silico data with known ground truth isolates the causal effect of outliers on normalization performance.

## Related tools

- **Metanorm** (R package implementing the five normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) applied to each simulated, outlier-contaminated feature) — https://github.com/UGent-LIMET/Metanorm
- **R** (Language in which the drift simulation, outlier injection, 200-run loop, and MSPE computation are executed (R >= 4.4.0 required)) — https://github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Installed via pak::pak("UGent-LIMET/Metanorm") to call metanorm() with model = 'QC-RLSC' | 'QC-RSC' | 'rLOESS' | 'rGAM' | 'tGAM' on the simulated matrix) — https://github.com/UGent-LIMET/Metanorm

## Examples

```
normdat <- metanorm(simdata, model = "tGAM", type = metanorm.qc, batch = batch)
```

## Evaluation signals

- mspe_estimates.csv contains exactly 40 MSPE rows covering all method-by-QC-density-by-outlier-count combinations
- Robust methods (rLOESS, rGAM, tGAM) show stable (comparatively unaffected) MSPE as the number of injected outliers increases, while QC-RLSC and QC-RSC degrade
- No injected outlier lies within the first or last five observations of the simulated series
- Each MSPE value is the median over 200 simulation runs, not a single run
- Both QC densities (one and two QC samples per 10 biological samples) are represented for every method and outlier count

## Limitations

- The simulation design could be enhanced to better mimic experimental metabolomics data (e.g., resampling strategies, cross-metabolite correlations); the current design does not capture cross-metabolite correlations
- Outlier treatment here is feature-level; multivariate outlier detection is not implemented and may improve results
- Multivariate methods (SERFF, TIGER) and a broader set of normalization methods are not benchmarked in this design
- tGAM outperformed on experimental data, but simulated-data performance was comparable between rLOESS, rGAM, and tGAM, so simulation alone cannot resolve fine-grained method differences

## Evidence

- [readme] package install/usage: [evidence span withheld — anchor status: external_doc]
