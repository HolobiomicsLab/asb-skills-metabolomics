---
name: prediction-error-aggregation-across-simulation-runs
description: Use when benchmarking normalization methods on in-silico metabolomics
  data with known ground-truth drift and injected outliers, and per-run squared prediction
  errors must be aggregated into median estimates.
when_to_use_negative:
- The data are real experimental metabolomics measurements without a known drift-free
  ground truth — squared prediction error against truth cannot be computed, and the
  paper instead uses replicate concordance, drift and batch-effect reduction on such
  data.
- The goal is to proxy normalization quality by reduction of the relative standard
  deviation of QC samples or increased numbers of differentially abundant metabolites
  — the paper explicitly flags these as common benchmarking pitfalls.
- Only a single method or a single outlier setting is being evaluated — cross-method
  comparison of degradation with increasing outlier count (e.g. QC-RLSC deteriorating
  while robust methods degrade gracefully) is the point of the aggregation.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metanorm
  role: metanorm function (model = "QC-RLSC", "rLOESS", "rGAM", or "tGAM") normalizes
    each simulated run so that prediction errors against ground truth can be computed
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Environment (version >= 4.4.0) in which the 200-run simulation, outlier injection,
    and median-error aggregation are executed
  repo: https://github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Package providing the metanorm normalization implementation and example dataset
    used to benchmark across methods
  repo: https://github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:ec884d0d3a8c058c238a8de4e4e5c11f7b6aeb7f577d0d63a06b61eca2de28ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm R package
  - R
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# prediction-error-aggregation-across-simulation-runs

## Summary

Aggregates per-run squared prediction errors of normalized metabolomics signals against a known drift-free ground truth into median estimates across 200 simulated runs, benchmarking QC-RLSC, QC-RSC, rLOESS, rGAM and tGAM under varying outlier counts and QC densities.

## When to use

Use when benchmarking normalization methods on in-silico metabolomics data with known ground-truth drift and injected outliers, and per-run squared prediction errors must be aggregated into median estimates.

## When NOT to use

- The data are real experimental metabolomics measurements without a known drift-free ground truth — squared prediction error against truth cannot be computed, and the paper instead uses replicate concordance, drift and batch-effect reduction on such data.
- The goal is to proxy normalization quality by reduction of the relative standard deviation of QC samples or increased numbers of differentially abundant metabolites — the paper explicitly flags these as common benchmarking pitfalls.
- Only a single method or a single outlier setting is being evaluated — cross-method comparison of degradation with increasing outlier count (e.g. QC-RLSC deteriorating while robust methods degrade gracefully) is the point of the aggregation.

## Inputs

- 200 independent simulation runs of drift data: simulated compounds following a sine-based drift signal with added Gaussian noise (sd 0.3)
- Outlier scenarios: 0, 1, 2, 3, and 4 feature-specific outliers injected per run at random non-edge positions (deviations of 20-100% or more)
- QC sample structure at densities of one and two QCs per 10 samples, remainder biological samples
- Normalized signal matrices produced per method via metanorm (model = "QC-RLSC", "rLOESS", "rGAM", or "tGAM")

## Outputs

- Per-method, per-compound, per-run squared prediction errors against the drift-free ground truth
- Median squared prediction error estimates per method and outlier setting, across 200 runs
- Results table organized like the panels of Fig. 1b and Supplementary Figs. S2-S4 (one and two QCs per 10 samples)

## How to apply

For every combination of outlier count (0-4 feature-specific outliers injected at random non-edge positions with deviations of 20-100% or more) and QC density (one or two QCs per 10 samples), normalize the simulated sine-based drift data (Gaussian noise, sd 0.3) with each candidate method via the metanorm function (model = "QC-RLSC", "rLOESS", "rGAM", or "tGAM"). For each method, compound, and run, compute the squared prediction error of the normalized signal against the known drift-free ground truth. Aggregate these errors to the median across the 200 independent runs — the median is chosen to be robust, mirroring the outlier-robustness being evaluated. Assemble the median squared prediction error per method and outlier setting into a results table structured like the panels of Fig. 1b and Supplementary Figs. S2-S4. Care should be taken to assure simulation neutrality, i.e. not simulating from a fitted model of one of the benchmarked methods.

## Related tools

- **metanorm** (metanorm function (model = "QC-RLSC", "rLOESS", "rGAM", or "tGAM") normalizes each simulated run so that prediction errors against ground truth can be computed) — https://github.com/UGent-LIMET/Metanorm
- **R** (Environment (version >= 4.4.0) in which the 200-run simulation, outlier injection, and median-error aggregation are executed) — https://github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Package providing the metanorm normalization implementation and example dataset used to benchmark across methods) — https://github.com/UGent-LIMET/Metanorm

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- For each method and outlier setting, the estimated median squared prediction error is finite and reproducible across the 200 runs.
- Relative comparisons mirror the qualitative pattern of Fig. 1b and Supplementary Figs. S2-S4: robust methods (rLOESS, rGAM, tGAM) degrade gracefully with increasing outlier count while QC-RLSC deteriorates.
- The aggregation uses the median (not the mean) across runs, so injected outliers in individual runs do not dominate the benchmark estimate.
- Errors are computed against the known drift-free ground truth per method, compound, and run — not against raw or pre-normalization signals.
- Results are reported separately for both QC densities (one and two QCs per 10 samples).

## Limitations

- In benchmarking, care should be taken to assure neutrality, i.e. not potentially favor any of the benchmarked methods, which can arise from simulating from a fitted model of one of them.
- Performance on simulated data was comparable between rLOESS, rGAM, and tGAM, so this benchmark alone does not reproduce the tGAM superiority observed on experimental datasets.
- Better mimicking of experimental data (e.g. resampling from observed data accounting for cross-metabolite correlations) and evaluation of a broader set of methods (e.g. SERFF, TIGER) remain future work.

## Evidence

- [methods] For each method, compound, and run, compute the squared prediction error of the normalized signal against the known drift-free ground truth and aggregate to the median across runs.: [evidence span withheld — anchor status: not_found]
- [results] The simulation protocol applies Gaussian noise (sd 0.3) to a sine-based drift pattern, injects 0-4 feature-specific outliers at random non-edge positions: [evidence span withheld — anchor status: not_found]
- [discussion] in silico simulation studies where the ground truth is known36. Such an approach was incorporated in this study: "in silico simulation studies where the ground truth is known36. Such an approach was incorporated in this study"
- [discussion] In benchmarking, care should be taken to assure neutrality, i.e. not potentially favor any of the benchmarked methods which can arise from, e.g., simulating from a fitted model from one of the: [evidence span withheld — anchor status: not_found]
