---
name: parameter-stability-visualization
description: Use when you must decide how many samples are sufficient for stable parameter
  estimation, by tabulating CV across random subsets and plotting CV vs. sample size
  against an acceptance threshold (e.g. CV <= 0.1).
when_to_use_negative:
- The parameter cannot be recomputed automatically per subsample — e.g. AutoTuner's
  Group difference parameter is estimated through a non-automatable cross-sample comparison,
  so its robustness cannot be checked this way.
- Input is already a processed feature table rather than raw data — AutoTuner requires
  raw converted instrument files (.mzML, .mzXML, .CDF).
- Fewer than 3 samples are available, since AutoTuner requires at least 3 samples
  of raw data as input.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3862
tools:
- name: AutoTuner
  role: Runs on each random subsample to obtain single-step statistical-inference
    estimates of the seven continuous centWave peak-picking parameters used by MZmine2
    and XCMS
  repo: https://github.com/KujawinskiLaboratory/Autotuner
- name: R
  role: Environment for loading raw samples, drawing random subsamples, computing
    coefficient of variation per parameter/subsample size, and producing the CV-vs-samples
    plot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:34932e634c5c1b5954da785c97978e9e573c8eaa479f3f0eb1f2ce6f80e3c2c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KujawinskiLaboratory/Autotuner
  tools:
  - AutoTuner
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3862
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# parameter-stability-visualization

## Summary

Quantifies the stability of automatically estimated data-processing parameters by running repeated estimation on random subsamples and plotting coefficient of variation (CV) against subsample size to find the minimum number of samples needed for stable estimates.

## When to use

Use when you must decide how many samples are sufficient for stable parameter estimation, by tabulating CV across random subsets and plotting CV vs. sample size against an acceptance threshold (e.g. CV <= 0.1).

## When NOT to use

- The parameter cannot be recomputed automatically per subsample — e.g. AutoTuner's Group difference parameter is estimated through a non-automatable cross-sample comparison, so its robustness cannot be checked this way.
- Input is already a processed feature table rather than raw data — AutoTuner requires raw converted instrument files (.mzML, .mzXML, .CDF).
- Fewer than 3 samples are available, since AutoTuner requires at least 3 samples of raw data as input.

## Inputs

- Raw mass-spectrometry sample files (.mzML, .mzXML, or .CDF) from the dataset under study (e.g. MTBLS157 culture samples)
- AutoTuner-compatible sample metadata spreadsheet with sample-name and experimental-factor columns
- List of subsample sizes to sweep (e.g. 3 to 9 samples) and number of random draws per size (e.g. 55)

## Outputs

- Table of per-run parameter estimates and CV values per parameter per subsample size (e.g. monte_carlo_cv_results.csv)
- Plot of CV versus number of samples per parameter (e.g. cv_vs_samples.png)
- Minimum sufficient sample size per parameter against the CV <= 0.1 acceptance criterion

## How to apply

For each experimental subset (e.g. the MTBLS157 culture dataset, 11 samples per subset), draw many random subsamples spanning a range of sizes (3 to 9 samples in the reference study; 55 draws per size per subset). Run AutoTuner on every drawn subsample — 2695 single-step statistical-inference runs in the reference study — collecting its estimates of the seven continuous centWave peak-picking parameters. For each parameter and each subsample size, compute the coefficient of variation across replicate draws, tabulate per-run estimates into a results table (e.g. monte_carlo_cv_results.csv), and plot CV versus number of samples per parameter (e.g. cv_vs_samples.png). Apply the acceptance criterion: a subsample size is sufficient when CV falls below 0.1 for most parameters; in the reference study 9 samples sufficed, supporting the recommendation of 9–12 samples for culture and community datasets.

## Related tools

- **AutoTuner** (Runs on each random subsample to obtain single-step statistical-inference estimates of the seven continuous centWave peak-picking parameters used by MZmine2 and XCMS) — https://github.com/KujawinskiLaboratory/Autotuner
- **R** (Environment for loading raw samples, drawing random subsamples, computing coefficient of variation per parameter/subsample size, and producing the CV-vs-samples plot)

## Examples

```
BiocManager::install(version='devel'); BiocManager::install("Autotuner")
```

## Evaluation signals

- All 7 continuous centWave parameters (Group difference excluded) have a tabulated CV for every subsample size in the sweep (3–9).
- CV curves decrease monotonically (reported as decreasing with sample number) as subsample size grows.
- At the 9-sample point, CV values are below 0.1 for most parameters, matching the reported acceptance criterion.
- Total number of AutoTuner runs matches the design matrix (e.g. 7 subsets x 55 draws x 7 sizes = 2695 runs).
- Output files exist with per-parameter rows and the CV-vs-samples figure renders one trace per parameter.

## Limitations

- The Group difference parameter could not be checked because it is estimated through a non-automatable cross-sample comparison during the TIC peak detection step.
- AutoTuner itself is not parallelized, so thousands of Monte Carlo runs are serial and computationally expensive.
- AutoTuner requires R 3.6 or greater and needs the sliding-window analysis stage supervised by user input.

## Evidence

- [abstract] We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection: "We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection, and found that AutoTuner generated similar parameter estimates from random subsets of samples."
- [discussion] For most parameters, 9 samples suffice: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [discussion] Recommend 9 to 12 samples: "we recommend the use of 9 to 12 samples to generate estimates in culture and community datasets, respectively."
- [discussion] Group difference not automatable: "We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison"
- [readme] AutoTuner raw data input formats: [evidence span withheld — anchor status: external_doc]
