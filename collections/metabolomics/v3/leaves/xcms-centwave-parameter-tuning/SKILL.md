---
name: xcms-centwave-parameter-tuning
description: Use when processing raw untargeted metabolomics mzML/mzXML/CDF data (qTOF,
  orbitrap, or FT-ICR) into a features table and no dataset-specific centWave parameter
  set exists, making manual tuning prohibitively time consuming.
when_to_use_negative:
- Input is already a features table or peaks list — parameter optimization is performed
  on raw data, not on processed feature tables
- Retention time correction parameters must be estimated from EIC peaks compared across
  samples — AutoTuner's divide-and-conquer approach comes at a loss of cross-sample
  EIC peak comparison
- Feature identification is the goal — identifying each distinct feature was beyond
  the scope of this study
tools:
- name: AutoTuner
  role: R package that performs the single-step centWave parameter estimation on raw
    data
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: XCMS
  role: Downstream data processing tool whose centWave peak-picking and grouping parameters
    are tuned
- name: MZmine2
  role: Alternative downstream tool that uses the same centWave peak-picking algorithm
    with tunable parameters
- name: IPO
  role: Iterative isotopologue parameter optimization baseline that is orders of magnitude
    slower; used as a runtime comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:64206259ca135b76ea252a661ff9b9a3c860c433401fff58af765fee480e44e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - IPO
  - MZmine2
  - R
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# xcms-centwave-parameter-tuning

## Summary

A method for dataset-specific tuning of centWave peak-picking parameters (used by XCMS and MZmine2) for untargeted metabolomics, exemplified by AutoTuner, which estimates seven continuous centWave parameters plus a grouping parameter in a single statistical-inference step on raw data instead of iterating over parameter combinations.

## When to use

Use when processing raw untargeted metabolomics mzML/mzXML/CDF data (qTOF, orbitrap, or FT-ICR) into a features table and no dataset-specific centWave parameter set exists, making manual tuning prohibitively time consuming.

## When NOT to use

- Input is already a features table or peaks list — parameter optimization is performed on raw data, not on processed feature tables
- Retention time correction parameters must be estimated from EIC peaks compared across samples — AutoTuner's divide-and-conquer approach comes at a loss of cross-sample EIC peak comparison
- Feature identification is the goal — identifying each distinct feature was beyond the scope of this study

## Inputs

- Raw untargeted metabolomics files in .mzML, .mzXML, or .CDF format (at least 3 samples) from qTOF, orbitrap, or FT-ICR mass analyzers
- Spreadsheet with at least two columns: sample-name column matching the raw data files and an experimental-factor column

## Outputs

- Optimized values for seven continuous centWave peak-picking parameters plus one key XCMS grouping parameter, ready for XCMS or MZmine2 data processing
- Downstream features table after peak-picking and retention time correction using the estimated parameters

## How to apply

Collect at least 3 raw samples converted to an open format (.mzML, .mzXML, or .CDF) plus a metadata spreadsheet with two columns: one matching sample names and one describing experimental factors. Install AutoTuner from BioConductor (requires R 3.6+) and run its two-stage workflow: a TIC-based sliding-window peak detection (the first portion requires user input, and the Group difference parameter is estimated via a non-automatable cross-sample comparison), followed by parameter sweeping that estimates seven continuous centWave parameters and one key XCMS grouping parameter. This reduces the combinatorial search from at least 24*5^8 candidate parameter combinations to 40, which is why it is roughly 100-1000X faster than iterative optimization with IPO (days versus minutes). Use 9 samples for culture datasets (12 for community datasets); 9 samples yielded parameter estimates with CV values less than 0.1 for most parameters. The mzDiff parameter is not optimized—apply a sensitivity analysis if needed, since distinct mzDiff values had minimal effect on the returned feature table.

## Related tools

- **AutoTuner** (R package that performs the single-step centWave parameter estimation on raw data) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **XCMS** (Downstream data processing tool whose centWave peak-picking and grouping parameters are tuned)
- **MZmine2** (Alternative downstream tool that uses the same centWave peak-picking algorithm with tunable parameters)
- **IPO** (Iterative isotopologue parameter optimization baseline that is orders of magnitude slower; used as a runtime comparison)

## Examples

```
BiocManager::install(version='devel'); BiocManager::install("Autotuner")
```

## Evaluation signals

- Monte Carlo robustness check: rerun parameter selection on random subsets of samples and verify similar parameter estimates (with 9 samples, CV < 0.1 for most parameters)
- Runtime benchmark: CPU-count-adjusted speed ratio vs IPO (AutoTuner on 1 CPU vs IPO on 8 CPUs) should fall in the reported ~100-1000X range, consistent with Table 3
- Verify that seven continuous centWave parameters plus one grouping parameter are returned, with mzDiff excluded from optimization
- Sensitivity analysis: vary mzDiff and confirm minimal effect on the returned feature table
- Manual inspection of raw data as a quality-control check for the data generation steps leading up to the analysis

## Limitations

- AutoTuner is not parallelized; parallelization to reduce computation time is a possible future improvement
- The Group difference parameter cannot be automatically robustness-checked because it is estimated through a non-automatable cross-sample comparison
- The sliding window analysis in the first portion of AutoTuner requires user input
- mzDiff is the one continuous centWave parameter not optimized by AutoTuner (only a sensitivity analysis is performed)
- The paucity of size-validated metabolome datasets precludes further evaluation of the feature number comparison

## Evidence

- [abstract] very fast with ~100-1000X speed improvement: "it is scalable, highly robust, and very fast (∼100-1000X speed improvement from other algorithms going from days to minutes)."
- [intro] seven continuous centWave parameters for XCMS and MZmine2: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS"
- [discussion] parameter search space reduced from 24*5^8 combinations to 40: "reduces the total combinations of available centWave parameters from at least 24* 58 possible choices of parameters to 40."
- [readme] input requirements: open raw formats and metadata spreadsheet: [evidence span withheld — anchor status: external_doc]
