---
name: metabolomics-drift-simulation
description: Use when you need ground-truth verification of normalization performance
  (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) under varying outlier counts (0/1/2/4) and
  QC densities (1 or 2 QC per 10 biological samples), avoiding QC-RSD-style benchmarking
  pitfalls.
when_to_use_negative:
- Real experimental data with no known ground-truth drift — use replicate concordance,
  pre/post-normalization plots, and PCA score plots instead of MSPE benchmarking
- Benchmarking multivariate normalization methods (e.g., SERFF, TIGER) — the simulation
  treats each feature independently at the feature level
- Data already normalized, or when the goal is downstream differential abundance testing
  rather than normalization-method evaluation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: Metanorm R package
  role: Supplies the five normalization models benchmarked on the simulated drift
    ('QC-RLSC', 'QC-RSC', 'rLOESS', 'rGAM', 'tGAM') via the metanorm() function
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >= 4.4.0) in which the drift simulation, outlier
    injection, and MSPE aggregation are implemented
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:ae8821477f31de5d226d89cf6e19abe94b01996e9f6dea6f38f84f070be26179
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-drift-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An in silico benchmark that simulates a single-feature metabolomics drift pattern with known ground truth, contaminates it with controlled outliers, and scores Metanorm normalization methods by median squared prediction error (MSPE) at QC indices. It causally isolates how outlier count and QC-sample density affect normalization accuracy.

## When to use

Use when you need ground-truth verification of normalization performance (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) under varying outlier counts (0/1/2/4) and QC densities (1 or 2 QC per 10 biological samples), avoiding QC-RSD-style benchmarking pitfalls.

## When NOT to use

- Real experimental data with no known ground-truth drift — use replicate concordance, pre/post-normalization plots, and PCA score plots instead of MSPE benchmarking
- Benchmarking multivariate normalization methods (e.g., SERFF, TIGER) — the simulation treats each feature independently at the feature level
- Data already normalized, or when the goal is downstream differential abundance testing rather than normalization-method evaluation

## Inputs

- Simulated single-feature intensity series (1000 samples per run) generated as intensity_i = 20 + sin(x_i) + epsilon_i, epsilon ~ N(0, 0.3^2)
- 84 QC intensities per run (two QC per 10 biological samples), drawn from a sine function offset by 20
- Outlier specification: count (0, 1, 2, or 4), random locations excluding the first and last five observations, and magnitude rules (-20-30%, -20-100%, -100%/+200% of simulated intensity)
- Expected (ground-truth) drift pattern for error computation

## Outputs

- mspe_estimates.csv: 40 rows of median squared prediction errors covering 5 methods x 2 QC densities x 4 outlier counts
- Per-method predicted drift values at all QC indices per simulation run

## How to apply

In R, simulate drift for 1000 samples per run by evaluating intensity_i = 20 + sin(x_i) + epsilon_i with epsilon ~ N(0, 0.3^2) at x_i = 0/999, 5/999, ..., 4995/999, and draw 84 QC intensities (two per 10 biological samples) from a sine function offset by 20 to avoid negative values; split the series into two equal parts for two distinct drift patterns. For each of k = 200 simulation runs, add zero, one, two, or four outliers at random locations (excluding the first and last five observations), with magnitudes set by subtracting 20-30%, subtracting 20-100%, or subtracting 100% / adding 200% of the simulated intensity, drawing outlier parameters feature-specifically per run. Normalize each simulated feature with the five Metanorm methods using both one and two QC samples per 10 biological samples. At all QC indices, compute the squared difference between each method's predicted and the expected drift pattern, then take the median across the 200 runs to obtain the median squared prediction error (MSPE) per method. Assemble the 40 MSPE estimates (5 methods x 2 QC densities x 4 outlier counts) into mspe_estimates.csv.

## Related tools

- **Metanorm R package** (Supplies the five normalization models benchmarked on the simulated drift ('QC-RLSC', 'QC-RSC', 'rLOESS', 'rGAM', 'tGAM') via the metanorm() function) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >= 4.4.0) in which the drift simulation, outlier injection, and MSPE aggregation are implemented) — https://cloud.r-project.org/index.html

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- mspe_estimates.csv contains exactly 40 MSPE rows covering all method-by-QC-density-by-outlier-count combinations
- Robust methods (rLOESS, rGAM, tGAM) show stable (comparatively unaffected) MSPE as the number of outliers increases
- Non-robust QC-based methods show deteriorating MSPE as outliers are added, consistent with the paper's claim that outliers deteriorate existing QC-based normalization
- MSPE is a median across 200 simulation runs, computed only at QC indices against the expected drift pattern
- Results are directionally consistent with the paper's reported simulation results (Figures S2-S4)

## Limitations

- The simulation design does not mimic experimental metabolomics data (no resampling strategies or cross-metabolite correlations), as flagged by the authors as future work
- Outlier treatment is at the feature level; multivariate outlier detection is not implemented
- Outlier placement excludes the first and last five observations, so edge-of-run behavior is not stress-tested
- Automated outlier detection and removal prior to normalization is nontrivial and is not provided as an alternative within this skill

## Evidence

- [methods] simulate drift for 1000 samples per run by evaluating intensity_i = 20 + sin(x_i) + epsilon_i: [evidence span withheld — anchor status: not_found]
- [methods] outlier count and placement rule: [evidence span withheld — anchor status: not_found]
- [methods] MSPE computation: [evidence span withheld — anchor status: not_found]
- [results] validation schema: [evidence span withheld — anchor status: not_found]
- [readme] package scope: [evidence span withheld — anchor status: external_doc]
- [abstract] robustness claim: [evidence span withheld — anchor status: not_found]
