---
name: lc-ms-drift-simulation-data-generation
description: Use when you must benchmark QC-based LC-MS normalization methods (QC-RLSC,
  rLOESS, rGAM, tGAM) on simulated data with known drift ground truth and injected
  outliers, scoring median squared prediction error.
when_to_use_negative:
- You need to normalize a real experimental LC-MS dataset for downstream analysis
  — call the metanorm normalization workflow directly instead of generating simulations
- No drift-free ground truth is available — the squared-prediction-error benchmark
  of this skill cannot be computed on unlabelled real data
- You want to simulate from observed data via resampling while accounting for cross-metabolite
  correlations — the article proposes this as a different, more realistic simulation
  strategy not covered by the sine + Gaussian-noise scheme
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metanorm
  role: metanorm() normalizes each simulated run with model = "QC-RLSC", "rLOESS",
    "rGAM" or "tGAM" so squared prediction error against the drift-free ground truth
    can be computed per scenario
  repo: github.com/UGent-LIMET/Metanorm
- name: pak
  role: installs the Metanorm package into R via pak::pak("UGent-LIMET/Metanorm")
    before running the simulation loop
- name: R
  role: runtime (version >= 4.4.0) hosting the 200-run simulation, outlier injection,
    QC-density layouts and median squared prediction error aggregation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:1ef3281a6bffe04b7fdf7cbc90381cbf7073b7a8dbaee32914179a2efc18b59c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lc-ms-drift-simulation-data-generation

## Summary

Generates in silico LC-MS runs with a known sine-based drift ground truth (Gaussian noise sd 0.3), injects 0–4 feature-specific outliers with 20–100%+ deviations at QC densities of one and two QCs per 10 samples, and benchmarks QC-RLSC, rLOESS, rGAM and tGAM by median squared prediction error across 200 runs.

## When to use

Use when you must benchmark QC-based LC-MS normalization methods (QC-RLSC, rLOESS, rGAM, tGAM) on simulated data with known drift ground truth and injected outliers, scoring median squared prediction error.

## When NOT to use

- You need to normalize a real experimental LC-MS dataset for downstream analysis — call the metanorm normalization workflow directly instead of generating simulations
- No drift-free ground truth is available — the squared-prediction-error benchmark of this skill cannot be computed on unlabelled real data
- You want to simulate from observed data via resampling while accounting for cross-metabolite correlations — the article proposes this as a different, more realistic simulation strategy not covered by the sine + Gaussian-noise scheme

## Inputs

- Simulated drift matrix per run (compounds × samples): sine-based drift + Gaussian noise (sd 0.3), paired with its known drift-free ground truth
- Outlier scenario specification: 0–4 feature-specific outliers per scenario at random non-edge positions, deviations of 20–100% or more
- Sample-type vector marking QC positions at densities of one and two QCs per 10 samples, remaining samples biological
- 200 independent simulation runs
- Metanorm R package in an R >= 4.4.0 environment

## Outputs

- Normalized signal matrices per method (QC-RLSC, rLOESS, rGAM, tGAM), outlier count (0–4) and QC density
- Median squared prediction error table per method × outlier setting, organized like the panels of Fig. 1b and Supplementary Figs. S2–S4 (one and two QCs per 10 samples)
- Validation record confirming the median squared prediction error is finite and reproducible across the 200 runs

## How to apply

In R (>= 4.4.0) with the Metanorm package installed via pak::pak("UGent-LIMET/Metanorm"), generate 200 independent runs in which each simulated compound follows a sine-based drift signal with added Gaussian noise (sd 0.3), so the drift-free ground truth is known for every compound. For each run, inject 0, 1, 2, 3 and 4 feature-specific outliers at random non-edge positions with deviations of 20–100% or more, and lay out the run order with QC samples at two densities — one and two QCs per 10 samples — keeping the remaining samples biological. For every outlier-count × QC-density combination, normalize by calling metanorm(rawdata, model = "QC-RLSC", "rLOESS", "rGAM" or "tGAM", type = <QC/sample labels>). For each method, compound and run, compute the squared prediction error of the normalized signal against the known drift-free ground truth and aggregate to the median across runs. Assemble the median squared prediction errors per method and outlier setting into a results table organized like Fig. 1b and Supplementary Figs. S2–S4. Keep the simulation neutral: do not generate the drift from a fitted model of one of the benchmarked methods, as the article warns this can favor that method.

## Related tools

- **metanorm** (metanorm() normalizes each simulated run with model = "QC-RLSC", "rLOESS", "rGAM" or "tGAM" so squared prediction error against the drift-free ground truth can be computed per scenario) — github.com/UGent-LIMET/Metanorm
- **pak** (installs the Metanorm package into R via pak::pak("UGent-LIMET/Metanorm") before running the simulation loop)
- **R** (runtime (version >= 4.4.0) hosting the 200-run simulation, outlier injection, QC-density layouts and median squared prediction error aggregation)

## Examples

```
pak::pak("UGent-LIMET/Metanorm"); library(metanorm); normdat <- metanorm(simdata, model = "tGAM", type = metanorm.qc)
```

## Evaluation signals

- For each method and outlier setting the estimated median squared prediction error is finite and reproducible across the 200 runs
- Relative comparisons mirror the qualitative pattern of Fig. 1b and Supplementary Figs. S2–S4: robust methods (rLOESS, rGAM, tGAM) degrade gracefully with increasing outlier count while QC-RLSC deteriorates
- The scenario grid is complete: outlier counts {0,1,2,3,4} × two QC densities (one and two QCs per 10 samples), each evaluated over 200 runs
- Injected outliers sit at random non-edge positions with deviations of 20–100% or more, and drift is sine-based with Gaussian noise of sd 0.3
- Benchmark neutrality holds: the simulated drift is not generated from a fitted model of any benchmarked method

## Limitations

- The sine-based drift with Gaussian noise (sd 0.3) is a simplification; the article recommends better mimicking experimental metabolomics data by simulating from observed data using resampling strategies while accounting for cross-metabolite correlations
- Neutrality risk: simulating from a fitted model of one of the benchmarked methods can potentially favor that method
- tGAM carries a higher computational cost than rLOESS and rGAM; scaling the benchmark to 200 runs × outlier scenarios × QC densities multiplies this cost
- Feature-level outlier injection does not capture multivariate outlier structure, which the article flags as a possible future improvement

## Evidence

- [abstract] Benchmarked robust normalization methods: "We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them."
- [methods] R version requirement: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [methods] Package installation command: "pak::pak("UGent-LIMET/Metanorm")"
- [discussion] Benchmark neutrality requirement: "In benchmarking, care should be taken to assure neutrality, i.e., not potentially favor any of the benchmarked methods which can arise from, e.g., simulating from a fitted model from one of the"
- [discussion] Ground-truth in-silico simulation rationale: "in silico simulation studies where the ground truth is known36. Such an approach was incorporated in this study"
- [readme] tGAM vs rLOESS/rGAM speed trade-off: "tGAM is recommended due to its superior robustness, but rGAM and rLOESS are faster."
- [discussion] tGAM computational cost caveat: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM; however, normalization remains computationally feasible, completing within minutes on current"
