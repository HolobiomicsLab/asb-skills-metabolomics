---
name: outlier-injection-robustness-assessment
description: Use when benchmarking outlier robustness of QC-based LC-MS metabolomics
  normalization methods on simulated data where ground-truth drift is known and outliers
  can be injected at controlled counts and magnitudes.
when_to_use_negative:
- Input is real experimental data without known ground truth - drift-corrected output
  cannot be scored by squared prediction error and requires experimental (replicate/batch-effect
  based) evaluation instead
- Benchmarking where you would use proxies like reduction of the relative standard
  deviation of QC samples or increased numbers of differentially abundant metabolites
  - the paper flags these as common benchmarking pitfalls
- Simulating from a fitted model of one of the benchmarked methods - this breaks neutrality
  and can favor that method
tools:
- name: R
  role: Runtime environment; version >= 4.4.0 required for the simulation and normalization
    benchmark
- name: pak
  role: Installs the Metanorm package from GitHub via pak::pak("UGent-LIMET/Metanorm")
- name: metanorm
  role: Function used to normalize each simulated run with model = "QC-RLSC", "QC-RSC",
    "rLOESS", "rGAM", or "tGAM" so squared prediction error can be computed per method
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Provides the metanorm normalization function and the metanorm example dataset
    used to exercise the package before the simulation benchmark
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:f500c93eb93eafccc40f7b377e4c0538fb93df54f0dc0f4a3c026bfeb4842bfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - R
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# outlier-injection-robustness-assessment

## Summary

An in-silico benchmarking protocol that generates simulated metabolomics runs with known ground-truth signal drift, injects controlled numbers of feature-specific outliers, and scores QC-based normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) by median squared prediction error against the drift-free truth.

## When to use

Use when benchmarking outlier robustness of QC-based LC-MS metabolomics normalization methods on simulated data where ground-truth drift is known and outliers can be injected at controlled counts and magnitudes.

## When NOT to use

- Input is real experimental data without known ground truth - drift-corrected output cannot be scored by squared prediction error and requires experimental (replicate/batch-effect based) evaluation instead
- Benchmarking where you would use proxies like reduction of the relative standard deviation of QC samples or increased numbers of differentially abundant metabolites - the paper flags these as common benchmarking pitfalls
- Simulating from a fitted model of one of the benchmarked methods - this breaks neutrality and can favor that method

## Inputs

- Simulated numerical intensity matrix (rows = compounds, columns = samples) with sine-based drift plus Gaussian noise (sd 0.3)
- Known drift-free ground-truth signal per compound per run
- Sample-type vector marking QC positions at 1 and 2 QCs per 10 samples
- Outlier scenario specification: 0-4 injected feature-specific outliers at random non-edge positions with 20-100%+ deviations
- Metanorm model specification strings: "QC-RLSC", "QC-RSC", "rLOESS", "rGAM", "tGAM"

## Outputs

- Per-run squared prediction error of normalized vs. ground-truth signal for each method, compound, and run
- Median squared prediction error per method per outlier count (0-4) per QC density
- Results table organized like Fig. 1b and Supplementary Figs. S2-S4 (one and two QCs per 10 samples)

## How to apply

In R (>= 4.4.0) with the Metanorm package, generate 200 independent simulation runs per setting in which each simulated compound follows a sine-based drift signal with added Gaussian noise (sd 0.3), inserting QC samples at a density of one QC per 10 samples and repeating at two QCs per 10 samples. For each run, inject 0, 1, 2, 3, and 4 feature-specific outliers at random non-edge positions with deviations of 20-100% or more to create the outlier scenarios. For every combination of outlier count and QC density, apply each benchmarked method via the metanorm function with model = "QC-RLSC", "QC-RSC", "rLOESS", "rGAM", or "tGAM". For each method, compound, and run, compute the squared prediction error of the normalized signal against the known drift-free ground truth and aggregate to the median across runs. Assemble the median squared prediction error per method and outlier setting into a results table mirroring the panels of Fig. 1b and Supplementary Figs. S2-S4. Ensure simulation neutrality: do not simulate from a fitted model of one of the benchmarked methods, as this would favor that method.

## Related tools

- **R** (Runtime environment; version >= 4.4.0 required for the simulation and normalization benchmark)
- **pak** (Installs the Metanorm package from GitHub via pak::pak("UGent-LIMET/Metanorm"))
- **metanorm** (Function used to normalize each simulated run with model = "QC-RLSC", "QC-RSC", "rLOESS", "rGAM", or "tGAM" so squared prediction error can be computed per method) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Provides the metanorm normalization function and the metanorm example dataset used to exercise the package before the simulation benchmark) — github.com/UGent-LIMET/Metanorm

## Examples

```
pak::pak("UGent-LIMET/Metanorm"); library(metanorm); normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch)
```

## Evaluation signals

- For each method and outlier setting, the estimated median squared prediction error over the 200 runs is finite and reproducible
- Relative comparisons across outlier counts (0-4) mirror the qualitative pattern of Fig. 1b and Supplementary Figs. S2-S4: robust methods (rLOESS, rGAM, tGAM) degrade gracefully with increasing outlier count while QC-RLSC deteriorates
- Results are produced at both QC densities (one and two QCs per 10 samples) and organized per method per outlier count
- Simulation design passes the neutrality check: ground truth is not generated from a fitted model of any benchmarked method
- Outlier injections occur at random non-edge positions with the specified 20-100%+ deviations, and each scenario is replicated across all 200 runs

## Limitations

- Simulation realism can be improved by resampling from observed data while accounting for cross-metabolite correlations, better mimicking experimental metabolomics data
- Care must be taken to assure neutrality in benchmarking - simulating from a fitted model of one of the benchmarked methods can favor that method
- tGAM carries a higher computational cost than rLOESS and rGAM, though normalization completes within minutes for thousands of metabolites and hundreds of samples
- Performance on simulated data was comparable between rLOESS, rGAM, and tGAM, so simulation alone may not discriminate these robust methods - experimental datasets were needed to distinguish tGAM

## Evidence

- [results] Sine-based drift with Gaussian noise and outlier injection: [evidence span withheld — anchor status: not_found]
- [results] Squared prediction error aggregation: [evidence span withheld — anchor status: not_found]
- [intro] Outlier rationale for the benchmark: "the presence of occasional, disproportionately high or low metabolite intensities (outliers) can deteriorate the performance of existing normalization methods"
- [discussion] Ground-truth simulation approach: "in silico simulation studies where the ground truth is known36. Such an approach was incorporated in this study"
- [discussion] Benchmarking neutrality caveat: "care should be taken to assure neutrality, i.e., not potentially favor any of the benchmarked methods which can arise from, e.g., simulating from a fitted model from one of the"
- [readme] R version requirement: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
