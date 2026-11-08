---
name: qc-based-normalization-method-benchmarking
description: Use when selecting among QC-based drift-normalization methods (QC-RLSC,
  QC-RSC, rLOESS, rGAM, tGAM) and outlier robustness must be quantified on simulated
  data with known ground-truth drift.
when_to_use_negative:
- Benchmark quality would be judged by reduction of the QC relative standard deviation
  (RSD) or by an increased number of differentially abundant metabolites — the article
  names both as benchmarking pitfalls rather than valid benchmarks.
- You only have real experimental data without a known drift ground truth — the median
  squared prediction error requires in-silico data where ground truth is known; experimental
  datasets need the separate data-driven evaluation approach.
- The simulation would be generated from a fitted model of one of the benchmarked
  methods — this breaks the neutrality the article demands and can favor that method.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: Metanorm R package
  role: Supplies metanorm() to run each candidate normalization (model = 'QC-RLSC',
    'rLOESS', 'rGAM', 'tGAM') on every simulation cell, plus QCcheck/plotPCA diagnostics
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime (version >= 4.4.0) in which the 200-run simulation, outlier injection
    and median squared prediction error aggregation are scripted
- name: pak
  role: Installs Metanorm directly from GitHub via pak::pak("UGent-LIMET/Metanorm")
- name: rLOESS
  role: Robust candidate normalization method benchmarked via metanorm(model = 'rLOESS')
- name: rGAM
  role: Robust candidate normalization method benchmarked via metanorm(model = 'rGAM')
- name: tGAM
  role: Robust candidate normalization method benchmarked via metanorm(model = 'tGAM');
    article's proposed default
- name: QC-RLSC
  role: Existing (non-robust) QC-based method benchmarked as the degrading baseline
    via metanorm(model = 'QC-RLSC')
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:ea112245e98a5af19e3bbd46aacc4a90cbbd492e7e485c0edf239332516ee2c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - QC-RLSC
  - R
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# qc-based-normalization-method-benchmarking

## Summary

Benchmark QC-based LC-MS metabolomics normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) on in-silico data with known sine-based drift and injected outliers, scoring each method by median squared prediction error over 200 simulated runs to quantify outlier robustness.

## When to use

Use when selecting among QC-based drift-normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) and outlier robustness must be quantified on simulated data with known ground-truth drift.

## When NOT to use

- Benchmark quality would be judged by reduction of the QC relative standard deviation (RSD) or by an increased number of differentially abundant metabolites — the article names both as benchmarking pitfalls rather than valid benchmarks.
- You only have real experimental data without a known drift ground truth — the median squared prediction error requires in-silico data where ground truth is known; experimental datasets need the separate data-driven evaluation approach.
- The simulation would be generated from a fitted model of one of the benchmarked methods — this breaks the neutrality the article demands and can favor that method.

## Inputs

- In-silico raw intensity matrix (rows = compounds, columns = samples) carrying a sine-based drift signal plus Gaussian noise (sd 0.3)
- Sample-type vector marking QC vs biological-sample positions at 1 or 2 QCs per 10 samples (the metanorm 'type' argument)
- Outlier injection plan: 0-4 feature-specific outliers at random non-edge run positions with 20-100%+ magnitude deviations
- Drift-free ground-truth signal per compound per run, used as reference for the squared prediction error

## Outputs

- Median squared prediction error per method x outlier count (0-4) x QC density, aggregated over 200 runs, formatted like Fig. 1b / Supplementary Figs. S2-S4
- Per-run normalized data matrices for each benchmarked normalization method

## How to apply

Generate ~200 independent runs in R (>= 4.4.0) in which each simulated compound follows a sine-based drift signal with added Gaussian noise (sd 0.3), inserting QC samples at one QC per 10 samples and repeating at two QCs per 10 samples. For each run, inject 0, 1, 2, 3, and 4 feature-specific outliers at random non-edge positions with deviations of 20-100% or more. For every outlier-count x QC-density combination, normalize with each candidate via the Metanorm package's metanorm() call with model = 'QC-RLSC', 'rLOESS', 'rGAM', or 'tGAM' (QC-RSC is also implemented alongside QC-RLSC). Per method, compound, and run, compute the squared prediction error of the normalized signal against the drift-free ground truth and aggregate to the median across the 200 runs. Keep the simulation neutral — the article warns not to simulate from a fitted model of one of the benchmarked methods — and assemble the median squared prediction errors into a results table organized like Fig. 1b and Supplementary Figs. S2-S4 (one panel per QC density). Interpret robustness as the robust methods (rLOESS/rGAM/tGAM) degrading gracefully with outlier count while QC-RLSC deteriorates, and avoid the flagged benchmarking pitfalls of using QC RSD reduction or counts of differentially abundant metabolites as proxies.

## Related tools

- **Metanorm R package** (Supplies metanorm() to run each candidate normalization (model = 'QC-RLSC', 'rLOESS', 'rGAM', 'tGAM') on every simulation cell, plus QCcheck/plotPCA diagnostics) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime (version >= 4.4.0) in which the 200-run simulation, outlier injection and median squared prediction error aggregation are scripted)
- **pak** (Installs Metanorm directly from GitHub via pak::pak("UGent-LIMET/Metanorm"))
- **rLOESS** (Robust candidate normalization method benchmarked via metanorm(model = 'rLOESS'))
- **rGAM** (Robust candidate normalization method benchmarked via metanorm(model = 'rGAM'))
- **tGAM** (Robust candidate normalization method benchmarked via metanorm(model = 'tGAM'); article's proposed default)
- **QC-RLSC** (Existing (non-robust) QC-based method benchmarked as the degrading baseline via metanorm(model = 'QC-RLSC'))

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch)
```

## Evaluation signals

- Every method x outlier count x QC density cell yields a finite median squared prediction error that is reproducible across the 200 runs.
- Relative pattern mirrors Fig. 1b and Supplementary Figs. S2-S4: robust methods (rLOESS, rGAM, tGAM) degrade gracefully as outliers rise from 0 to 4 while QC-RLSC deteriorates.
- On simulated data, rLOESS, rGAM and tGAM results should be comparable per the article's finding; large gaps among the three robust methods suggest a simulation or parameterization error.
- Neutrality check: the ground truth is not simulated from a fitted model of any benchmarked method.
- Results table is split into separate panels for the two QC densities (one and two QCs per 10 samples).

## Limitations

- Neutrality risk: simulating from a fitted model of one of the benchmarked methods can favor it and bias the comparison.
- The sine-based drift plus Gaussian-noise design may not mimic real experimental data; the authors suggest resampling from observed data while accounting for cross-metabolite correlations.
- Coverage is limited to the methods studied; a broader set (e.g., SERFF, TIGER) remains to be benchmarked.
- Because rLOESS, rGAM and tGAM perform comparably on simulated data, simulation alone cannot establish the recommended default — that came from experimental datasets where tGAM was strongest.
- Only feature-level outlier injection is modeled; multivariate outlier behavior is untested.

## Evidence

- [intro] Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant: "Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant"
- [discussion] in silico simulation studies where the ground truth is known36. Such an approach was incorporated in this study: "in silico simulation studies where the ground truth is known36. Such an approach was incorporated in this study"
- [discussion] care should be taken to assure neutrality, i.e., not potentially favor any of the benchmarked methods: "care should be taken to assure neutrality, i.e., not potentially favor any of the benchmarked methods"
- [discussion] Overall, the tGAM method exhibited the strongest performance across the experimental datasets, whereas performance on simulated data was comparable between rLOESS, rGAM, and tGAM.: "Overall, the tGAM method exhibited the strongest performance across the experimental datasets, whereas performance on simulated data was comparable between rLOESS, rGAM, and tGAM."
- [abstract] Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice.: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [readme] Metanorm supports robust metabolomics data normalization across scales and experimental designs.: "Metanorm supports robust metabolomics data normalization across scales and experimental designs."
