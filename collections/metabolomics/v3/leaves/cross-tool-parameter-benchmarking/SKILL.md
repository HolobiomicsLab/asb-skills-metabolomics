---
name: cross-tool-parameter-benchmarking
description: Use when two or more parameter-optimization tools (e.g., AutoTuner, IPO)
  have produced centWave parameter sets for the same raw LC-MS dataset and the analysis
  goal is to compare how each choice changes feature counts.
when_to_use_negative:
- The input is already a processed feature table rather than raw instrument data —
  parameter estimation must occur on raw data prior to analysis
- Fewer than 3 samples are available — AutoTuner requires at least 3 samples of raw
  data, and the paper recommends 9-12 samples for stable parameter estimates
- The question is feature identification/annotation rather than benchmarking parameter-selection
  methods; the article states identifying each distinct feature was beyond its scope
tools:
- name: AutoTuner
  role: 'Source of the first parameter set: single-step statistical-inference estimate
    of seven continuous centWave parameters plus a grouping parameter from raw data'
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: IPO
  role: 'Source of the comparison parameter set: iterative optimization of the same
    centWave parameters'
- name: XCMS
  role: Executes centWave peak detection, grouping, and retention-time correction
    under each parameter set to generate the feature tables being compared
- name: R
  role: Implementation environment; AutoTuner is an R/BioConductor package requiring
    R >= 3.6
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:25d1a09e528db118d6e581cf507ffc1c147b053e0866cc89190fdfe21b865f3b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - IPO
  - R
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-tool-parameter-benchmarking

## Summary

Benchmark metabolomics peak-picking parameter sets produced by competing selection tools (AutoTuner vs IPO) by running each parameter set through the identical XCMS centWave pipeline and quantifying unique and shared features in the resulting feature tables per ionization mode.

## When to use

Use when two or more parameter-optimization tools (e.g., AutoTuner, IPO) have produced centWave parameter sets for the same raw LC-MS dataset and the analysis goal is to compare how each choice changes feature counts.

## When NOT to use

- The input is already a processed feature table rather than raw instrument data — parameter estimation must occur on raw data prior to analysis
- Fewer than 3 samples are available — AutoTuner requires at least 3 samples of raw data, and the paper recommends 9-12 samples for stable parameter estimates
- The question is feature identification/annotation rather than benchmarking parameter-selection methods; the article states identifying each distinct feature was beyond its scope

## Inputs

- Raw untargeted metabolomics files in open formats (.mzML, .mzXML, or .CDF), split by ionization mode
- AutoTuner-derived centWave parameter set (7 continuous parameters plus XCMS grouping parameter)
- IPO-derived centWave parameter set for the same data
- Sample metadata spreadsheet with columns matching sample names and describing experimental factors

## Outputs

- One XCMS-processed feature table per parameter set per ionization mode (post QC filtering)
- feature_counts_mtbls157.csv: unique and shared feature counts per ionization mode comparing AutoTuner vs IPO parameters

## How to apply

Split the raw metabolomics files (e.g., the MetaboLights culture dataset MTBLS157) by positive and negative ionization mode. Estimate centWave peak-picking parameters with AutoTuner, which selects seven continuously valued centWave parameters plus a key XCMS grouping parameter, and with IPO, which estimates the same parameters iteratively. Run XCMS centWave peak detection, grouping, and retention-time correction once per parameter set on each ionization mode, holding the processing pipeline constant so that only the parameter source differs. Apply the paper's quality-control filtering to each resulting feature table, then count unique and shared features per ionization mode (AutoTuner vs IPO) and write the comparison to feature_counts_mtbls157.csv. The rationale is that no universal parameter set exists, so the effect of parameter choice on the feature table must be measured empirically rather than assumed.

## Related tools

- **AutoTuner** (Source of the first parameter set: single-step statistical-inference estimate of seven continuous centWave parameters plus a grouping parameter from raw data) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **IPO** (Source of the comparison parameter set: iterative optimization of the same centWave parameters)
- **XCMS** (Executes centWave peak detection, grouping, and retention-time correction under each parameter set to generate the feature tables being compared)
- **R** (Implementation environment; AutoTuner is an R/BioConductor package requiring R >= 3.6)

## Examples

```
BiocManager::install(version='devel'); BiocManager::install("Autotuner")
```

## Evaluation signals

- Check reported benchmarks against paper values: culture dataset positive mode AutoTuner 203 unique vs IPO 2606 unique features with 1022 shared; full-table totals 540 vs 3420 unique with 904 shared
- Confirm both feature tables were generated by the identical XCMS workflow (peak detection, grouping, retention-time correction) so differences are attributable to parameters alone
- Verify QC filtering was applied identically to both tables before counting features
- Verify unique + shared counts are internally consistent within each ionization mode (unique_A + shared and unique_IPO + shared cover the union)
- Confirm output CSV feature_counts_mtbls157.csv is written with per-ionization-mode rows

## Limitations

- IPO can be much slower: iterative parameter selection depends on high-performance computing resources and can take weeks, whereas AutoTuner achieves ~100-1000X speed improvement (days to minutes)
- Group difference parameter robustness could not be checked because it is estimated through a non-automatable cross sample comparison
- Paucity of size-validated metabolome datasets precludes further evaluation of the feature number comparison
- AutoTuner is not parallelized; the divide-and-conquer speed gain comes at a loss of comparing EIC peaks across samples for retention-time correction estimation

## Evidence

- [abstract] IPO comparison basis: "in comparison to isotopologue parameter optimization (IPO), the most commonly-used parameter selection tool"
- [intro] AutoTuner parameter scope: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS"
- [discussion] IPO accuracy caveat: "One possible explanation for the lower accuracy of IPO is that the peak-width of the missing standards was below the Minimum Peak-width parameter selected by IPO"
- [readme] AutoTuner input requirements: [evidence span withheld — anchor status: external_doc]
