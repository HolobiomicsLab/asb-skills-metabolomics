---
name: monte-carlo-resampling-analysis
description: Use when you have raw mass-spectrometry samples (e.g. MTBLS157 culture
  data) and need to determine how many samples are required for stable AutoTuner parameter
  estimates, via CV-vs-sample-size resampling.
when_to_use_negative:
- You want to check robustness of the Group difference parameter — it is estimated
  through a non-automatable cross sample comparison and could not be checked in the
  source study
- You already have a processed feature table — AutoTuner requires raw data converted
  from proprietary instrument formats (e.g. .mzML, .mzXML, .CDF)
- You have fewer than 3 samples per experimental condition — AutoTuner requires at
  least 3 raw-data samples for parameter estimation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: Generates the single-step statistical-inference estimates of the seven continuous
    centWave peak-picking parameters from each random subsample
  repo: https://github.com/KujawinskiLaboratory/Autotuner
- name: R
  role: Environment in which AutoTuner runs (R >= 3.6) and in which the Monte Carlo
    resampling, CV computation, tabulation, and plotting are performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:2674a13f4afa3a6807289b378cb581615bc7fec17bbdfda1d5bf76b030e9ea76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KujawinskiLaboratory/Autotuner
  tools:
  - AutoTuner
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# monte-carlo-resampling-analysis

## Summary

Assesses the robustness of AutoTuner's single-step centWave parameter estimates by repeatedly estimating parameters from random subsamples of raw metabolomics samples and computing the coefficient of variation (CV) as a function of subsample size.

## When to use

Use when you have raw mass-spectrometry samples (e.g. MTBLS157 culture data) and need to determine how many samples are required for stable AutoTuner parameter estimates, via CV-vs-sample-size resampling.

## When NOT to use

- You want to check robustness of the Group difference parameter — it is estimated through a non-automatable cross sample comparison and could not be checked in the source study
- You already have a processed feature table — AutoTuner requires raw data converted from proprietary instrument formats (e.g. .mzML, .mzXML, .CDF)
- You have fewer than 3 samples per experimental condition — AutoTuner requires at least 3 raw-data samples for parameter estimation

## Inputs

- Raw mass-spectrometry sample files (MTBLS157 culture dataset, 11 samples per subset; formats such as .mzML, .mzXML, or .CDF) in R
- Metadata spreadsheet matching raw data sample names to experimental factors (required AutoTuner input)
- Random subsample draws: 55 subsamples per subset per size, sizes 3 to 9 samples

## Outputs

- Per-run AutoTuner parameter estimates and CV values tabulated in monte_carlo_cv_results.csv
- Plot of CV versus number of samples for each parameter (cv_vs_samples.png)
- Verdict on the minimum sample count needed for CV <= 0.1 per parameter

## How to apply

Load raw MTBLS157 culture MS sample files into R and, for each of the 7 subsets, draw 55 random subsamples with sizes ranging from 3 to 9 samples. Run AutoTuner on every drawn subsample (2695 runs total) to obtain its single-step statistical-inference estimates of the seven continuous centWave peak-picking parameters used by MZmine2 and XCMS. For each parameter and each subsample size, compute the coefficient of variation across the replicate random draws, tabulate per-run estimates and CV values into monte_carlo_cv_results.csv, and plot CV versus number of samples per parameter as cv_vs_samples.png. Check results against the acceptance criterion that CV <= 0.1; in the source study, 9 samples proved sufficient for most parameters (9-12 samples recommended for culture and community datasets respectively). Exclude the Group difference parameter from robustness checking because it is estimated through a non-automatable cross-sample comparison during the TIC peak detection step.

## Related tools

- **AutoTuner** (Generates the single-step statistical-inference estimates of the seven continuous centWave peak-picking parameters from each random subsample) — https://github.com/KujawinskiLaboratory/Autotuner
- **R** (Environment in which AutoTuner runs (R >= 3.6) and in which the Monte Carlo resampling, CV computation, tabulation, and plotting are performed)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install(version='devel'); BiocManager::install("Autotuner")
```

## Evaluation signals

- CV versus sample-size trend decreases toward the acceptance threshold: CV <= 0.1 for most parameters at 9 samples
- Run count matches the design: 7 subsets x 55 random draws x sizes 3-9 = 2695 AutoTuner runs
- monte_carlo_cv_results.csv contains one row per run with per-run estimates of the seven continuous centWave parameters and CV values
- The Group difference parameter is absent from the robustness verdict (non-automatable, excluded by design)
- Estimates remain similar across random subsets (AutoTuner 'generated similar parameter estimates from random subsets of samples')

## Limitations

- Robustness of the Group difference parameter estimate could not be checked because it is estimated through a non-automatable cross-sample comparison
- AutoTuner is not parallelized, so 2695 subsample runs may be slow; parallelization is listed as a possible future improvement
- Subsample size is bounded by available samples (11 per subset in MTBLS157); 9-12 samples are recommended for culture and community datasets respectively

## Evidence

- [abstract] Monte Carlo robustness of parameter estimates: "We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection, and found that AutoTuner generated similar parameter estimates from random subsets of samples."
- [discussion] CV threshold and sample count: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [discussion] Group difference exclusion: "We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison"
- [discussion] Sample-size recommendation: "we recommend the use of 9 to 12 samples to generate estimates in culture and community datasets, respectively."
- [readme] AutoTuner input requirements: [evidence span withheld — anchor status: external_doc]
