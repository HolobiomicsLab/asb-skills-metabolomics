---
name: metabolomics-parameter-optimization
description: Use when starting untargeted metabolomics analysis from raw instrument
  files (.mzML, .mzXML, .CDF) and no dataset-specific centWave/peak-picking parameters
  exist yet; manual tuning is impractical or a universal parameter set is unavailable.
when_to_use_negative:
- Input is already a feature table or annotated data — parameter optimization on raw
  data is required before analysis to avoid noise inflation, and AutoTuner operates
  on raw files, not processed features
- The mzDiff parameter is the target — AutoTuner does not optimize it; only a sensitivity
  analysis showing minimal effect on the feature table is available
- Fully automated estimation of the Group difference parameter is needed — it is estimated
  through a non-automatable cross-sample comparison during TIC peak detection and
  requires user input
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: Performs the single-step statistical-inference parameter estimation on raw
    data; distributed as an R package through BioConductor
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: IPO
  role: Iterative isotopologue parameter optimization tool used as the slower baseline
    comparison (~100-1000X slower than AutoTuner)
- name: XCMS
  role: Downstream data processing tool whose centWave peak-picking parameters (and
    a key grouping parameter) are the targets being estimated
- name: MZmine2
  role: Alternative downstream tool that also uses the centWave algorithm with the
    same seven continuous parameters
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:52db066ec1ad6d3388886453acac2a9a7ba7d19e5ddc23b6cf2e486e0d7399d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - IPO
  - MZmine2
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# metabolomics-parameter-optimization

## Summary

Automatically select dataset-specific peak-picking parameters (centWave parameters used by MZmine2/XCMS plus a key XCMS grouping parameter) for untargeted metabolomics raw data, using AutoTuner's single-step statistical inference on raw mzML/mzXML/CDF files instead of slow iterative parameter sweeping.

## When to use

Use when starting untargeted metabolomics analysis from raw instrument files (.mzML, .mzXML, .CDF) and no dataset-specific centWave/peak-picking parameters exist yet; manual tuning is impractical or a universal parameter set is unavailable.

## When NOT to use

- Input is already a feature table or annotated data — parameter optimization on raw data is required before analysis to avoid noise inflation, and AutoTuner operates on raw files, not processed features
- The mzDiff parameter is the target — AutoTuner does not optimize it; only a sensitivity analysis showing minimal effect on the feature table is available
- Fully automated estimation of the Group difference parameter is needed — it is estimated through a non-automatable cross-sample comparison during TIC peak detection and requires user input

## Inputs

- Raw untargeted metabolomics data files (.mzML, .mzXML, or .CDF), at least 3 samples
- Metadata spreadsheet with two columns: sample name (matching raw files) and experimental factor description
- R environment version 3.6 or greater with the Autotuner package installed

## Outputs

- Estimated values for seven continuous centWave peak-picking parameters (used by MZmine2 and XCMS)
- A key XCMS grouping parameter (Group difference, estimated via cross-sample comparison)
- Parameter estimates with coefficient-of-variation values usable as inputs to XCMS/MZmine2 feature-table generation

## How to apply

Install the AutoTuner R package (>=R 3.6) via BiocManager::install('Autotuner'). Provide at least 3 raw samples converted from proprietary instrument formats (.mzML, .mzXML, or .CDF) plus a spreadsheet with two columns: sample-name matching and experimental factor descriptions. AutoTuner first runs a TIC peak detection / sliding window analysis (the Group difference parameter is estimated via a non-automatable cross-sample comparison requiring user input), then performs single-step statistical inference on raw data to estimate seven continuous centWave parameters used by MZmine2 and XCMS. This avoids iterative parameter sweeping, reducing combinations from at least 24*5^8 choices to 40 and yielding ~100-1000X speedup (days to minutes) over IPO. For robust estimates, run on 9 samples for culture datasets and 12 for community datasets; check coefficient of variation, expecting CV < 0.1 for most parameters. Manually inspect raw data afterward as a quality control check on data generation.

## Related tools

- **AutoTuner** (Performs the single-step statistical-inference parameter estimation on raw data; distributed as an R package through BioConductor) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **IPO** (Iterative isotopologue parameter optimization tool used as the slower baseline comparison (~100-1000X slower than AutoTuner))
- **XCMS** (Downstream data processing tool whose centWave peak-picking parameters (and a key grouping parameter) are the targets being estimated)
- **MZmine2** (Alternative downstream tool that also uses the centWave algorithm with the same seven continuous parameters)

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("Autotuner")
```

## Evaluation signals

- Speed check: runtime on a dataset such as MTBLS157 should show a CPU-count-adjusted ratio vs IPO in the ~100-1000X range (AutoTuner time * 1 vs IPO time * 8)
- Robustness check: parameter estimates from random sample subsets (Monte Carlo experiment) should be similar; for most parameters 9 samples should give CV values less than 0.1
- Parameter coverage check: seven continuous centWave parameters plus one grouping parameter are returned; mzDiff should be absent since it is not optimized
- Data check: fewer than 3 raw samples or a metadata sheet missing name/factor columns should fail, per README input requirements
- QC check: manual inspection of raw data is supported as a quality control step after estimation

## Limitations

- The Group difference parameter estimate's robustness could not be checked because it requires a non-automatable cross-sample comparison
- The sliding window analysis (first portion of AutoTuner) requires user input; a more sophisticated peak detection approach might eliminate this
- The divide-and-conquer approach does not compare EIC peaks across samples, so retention time correction algorithms cannot be estimated from it
- AutoTuner is not parallelized, though parallelization could reduce computation time by a factor of total CPUs used

## Evidence

- [abstract] speed-robustness finding: "it is scalable, highly robust, and very fast (∼100-1000X speed improvement from other algorithms going from days to minutes)"
- [readme] README input requirements: [evidence span withheld — anchor status: external_doc]
- [discussion] sample-size recommendation: "we recommend the use of 9 to 12 samples to generate estimates in culture and community datasets, respectively"
- [intro] single-step inference rationale: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values"
- [discussion] param space reduction: "reduces the total combinations of available centWave parameters from at least 24* 58 possible choices of parameters to 40"
