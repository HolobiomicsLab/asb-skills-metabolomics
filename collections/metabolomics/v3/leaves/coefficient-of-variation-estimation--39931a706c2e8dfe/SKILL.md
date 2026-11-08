---
name: coefficient-of-variation-estimation
description: Use when you have ≥3 raw mass-spectrometry samples (e.g. .mzML/.mzXML/.CDF)
  and need to quantify how stable AutoTuner-derived peak-picking parameter estimates
  are as sample count grows, to pick a minimum sample size.
when_to_use_negative:
- Input is already a feature table or processed data — AutoTuner estimates parameters
  from raw data, not from feature tables
- You want to validate the Group difference parameter — it is estimated through a
  non-automatable cross sample comparison and was explicitly excluded from robustness
  checking
- You have fewer than 3 samples, the minimum AutoTuner requires, so subsample-based
  CV estimation is not feasible
edam_operation: http://edamontology.org/operation_2945
tools:
- name: AutoTuner
  role: Run on every random subsample to obtain single-step statistical-inference
    estimates of the seven continuous centWave peak-picking parameters whose CV is
    computed
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Environment for running AutoTuner (R ≥ 3.6), drawing random subsamples, computing
    CVs, and generating result tables and plots
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:487b41425a132168661ee6fc681443261eda513b55e6b88d380ebdf13fe88a4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# coefficient-of-variation-estimation

## Summary

Assess the robustness of AutoTuner's single-step centWave parameter estimates by computing the coefficient of variation (CV) of each parameter across replicate random subsamples of raw metabolomics files, as a function of the number of samples used per estimate.

## When to use

Use when you have ≥3 raw mass-spectrometry samples (e.g. .mzML/.mzXML/.CDF) and need to quantify how stable AutoTuner-derived peak-picking parameter estimates are as sample count grows, to pick a minimum sample size.

## When NOT to use

- Input is already a feature table or processed data — AutoTuner estimates parameters from raw data, not from feature tables
- You want to validate the Group difference parameter — it is estimated through a non-automatable cross sample comparison and was explicitly excluded from robustness checking
- You have fewer than 3 samples, the minimum AutoTuner requires, so subsample-based CV estimation is not feasible

## Inputs

- Raw untargeted metabolomics sample files converted from proprietary formats (.mzML, .mzXML, or .CDF), at least 3 samples
- Sample metadata spreadsheet mapping sample names to experimental factors
- Randomly drawn subsamples of the raw files at increasing sizes (3 to 9 samples)

## Outputs

- Per-run AutoTuner parameter estimates across replicate subsamples
- Coefficient of variation per parameter per subsample size (e.g. monte_carlo_cv_results.csv)
- CV-versus-number-of-samples plot per parameter (e.g. cv_vs_samples.png)

## How to apply

From the raw MTBLS157 culture mass-spectrometry sample files, draw many random subsets (here 55 draws per subset) at successively larger sizes ranging from 3 to 9 samples, and run AutoTuner on every drawn subset to obtain its single-step statistical-inference estimates of the seven continuous centWave peak-picking parameters. For each parameter and each subsample size, compute the coefficient of variation (CV = SD/mean) of the estimate across the replicate draws. Tabulate the per-run estimates and CV values (e.g. monte_carlo_cv_results.csv) and plot CV versus number of samples per parameter (e.g. cv_vs_samples.png). Judge success against the acceptance criterion that CV ≤ 0.1 is reached; in this analysis 9 samples sufficed for most parameters, and the recommendation is 9 samples for culture datasets (12 for community datasets). Exclude the Group difference parameter from robustness checking, since it is estimated through a non-automatable cross sample comparison.

## Related tools

- **AutoTuner** (Run on every random subsample to obtain single-step statistical-inference estimates of the seven continuous centWave peak-picking parameters whose CV is computed) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Environment for running AutoTuner (R ≥ 3.6), drawing random subsamples, computing CVs, and generating result tables and plots) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("Autotuner")
```

## Evaluation signals

- All seven continuous centWave parameters reach CV ≤ 0.1 by ~9 samples per subsample (Group difference parameter excluded)
- Estimate variability decreases monotonically with the number of samples used per estimate
- Result table contains per-run estimates and CV values for every draw (2695 runs = 7 subsets × 55 draws × 7 sizes in the reference experiment)
- Each parameter estimate is computed on raw data in a single inference step, not by iterative parameter sweeps
- The final CV-vs-samples plot covers sizes from 3 to 9 samples per parameter

## Limitations

- The Group difference parameter could not be checked for robustness because it is estimated through a non-automatable cross sample comparison during the TIC peak detection step
- mzDiff, the one continuous centWave parameter not optimized by AutoTuner, was only assessed via a sensitivity analysis, not included in this CV analysis
- AutoTuner requires at least 3 raw samples and R version 3.6 or greater; it is not parallelized, so a large Monte Carlo design (thousands of runs) can be slow
- Robustness conclusions are dataset-specific: the 9-sample threshold is recommended for culture datasets (12 for community datasets) and may not transfer to other data types

## Evidence

- [abstract] We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection, and found that AutoTuner generated similar parameter estimates from random subsets of samples.: "We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection, and found that AutoTuner generated similar parameter estimates from random subsets of samples."
- [discussion] For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1.: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [discussion] We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison: "We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison"
- [readme] For input, AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF).: [evidence span withheld — anchor status: external_doc]
- [intro] The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values.: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
