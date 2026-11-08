---
name: raw-spectra-noise-filtering-and-peak-alignment
description: Use when starting from raw instrument files (.mzML, .mzXML, .CDF) from
  untargeted metabolomics experiments and you need a cross-sample aligned features
  table rather than per-file peak lists.
when_to_use_negative:
- Data are already processed into a features table — AutoTuner operates on raw data
  converted from proprietary instrument formats
- 'You need automated estimation of the Group difference parameter or mzDiff: Group
  difference is estimated by non-automatable cross-sample comparison during TIC peak
  detection, and mzDiff is only addressed via sensitivity analysis'
- Sample count is far below the recommended 9–12 samples (9 for culture, 12 for community
  datasets), giving CV > 0.1 for parameter estimates
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: Single-step statistical inference on raw data to estimate centWave peak-picking
    and grouping parameters; provided as an R package through BioConductor
  repo: https://github.com/KujawinskiLaboratory/Autotuner
- name: XCMS
  role: Executes centWave peak detection, grouping, and retention time correction
    using the AutoTuner-derived parameters to build the features table
- name: MZmine2
  role: Alternative data processing tool sharing the centWave peak-picking algorithm,
    whose 11 parameters AutoTuner tunes
- name: IPO
  role: Benchmark iterative parameter-optimization tool; AutoTuner achieves ~100–1000X
    speed improvement over it (days to minutes)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:b2c3a87a437ba620d7de26f970428d740465e4e61dc91f0d34f1ada8cd72ecac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KujawinskiLaboratory/Autotuner
  tools:
  - AutoTuner
  - BioConductor
  - IPO
  - MZmine2
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# raw-spectra-noise-filtering-and-peak-alignment

## Summary

Pipeline stage that processes raw untargeted metabolomics spectra (mzML/mzXML/CDF from qTOF, Orbitrap, or FT-ICR instruments) to remove noise and align mass-spectral peaks across samples, producing a features table whose quality depends on dataset-specific centWave parameters.

## When to use

Use when starting from raw instrument files (.mzML, .mzXML, .CDF) from untargeted metabolomics experiments and you need a cross-sample aligned features table rather than per-file peak lists.

## When NOT to use

- Data are already processed into a features table — AutoTuner operates on raw data converted from proprietary instrument formats
- You need automated estimation of the Group difference parameter or mzDiff: Group difference is estimated by non-automatable cross-sample comparison during TIC peak detection, and mzDiff is only addressed via sensitivity analysis
- Sample count is far below the recommended 9–12 samples (9 for culture, 12 for community datasets), giving CV > 0.1 for parameter estimates

## Inputs

- Raw untargeted metabolomics files (.mzML, .mzXML, or .CDF) from at least 3 samples (qTOF, Orbitrap, or FT-ICR mass analyzers)
- Sample metadata spreadsheet with one column matching sample names and one describing experimental factors
- R version 3.6 or greater with the Autotuner Bioconductor package and XCMS

## Outputs

- Dataset-specific AutoTuner parameter estimates (seven continuous centWave parameters plus Group difference)
- Per-sample peak lists (TIC-detected peaks)
- Cross-sample aligned features table with retention time correction
- mzDiff sensitivity summary quantifying feature-table differences across 0.001-increment mzDiff values

## How to apply

Convert proprietary instrument files to mzML, mzXML, or CDF, then run peak detection (centWave, used by XCMS and MZmine2, which requires tuning of 11 distinct parameters) followed by grouping and retention time correction to align peaks across samples. Optimize parameters first with AutoTuner: it performs statistical inference on raw data in a single step to estimate seven continuously valued centWave parameters plus the XCMS Group difference parameter, avoiding noise inflation within the feature table and reducing the parameter search from at least 24*5^8 combinations to ~40. Supplement AutoTuner with a sensitivity analysis for mzDiff, the only continuous centWave parameter not optimized, by comparing feature tables generated with mzDiff values varying in increments of 0.001. Validate parameters via a Monte Carlo check on random subsets of samples, and inspect raw data manually as a quality control step. The output feature table should contain fewer but more informative features than untuned processing.

## Related tools

- **AutoTuner** (Single-step statistical inference on raw data to estimate centWave peak-picking and grouping parameters; provided as an R package through BioConductor) — https://github.com/KujawinskiLaboratory/Autotuner
- **XCMS** (Executes centWave peak detection, grouping, and retention time correction using the AutoTuner-derived parameters to build the features table)
- **MZmine2** (Alternative data processing tool sharing the centWave peak-picking algorithm, whose 11 parameters AutoTuner tunes)
- **IPO** (Benchmark iterative parameter-optimization tool; AutoTuner achieves ~100–1000X speed improvement over it (days to minutes))

## Examples

```
BiocManager::install("Autotuner"); Autotuner <- runAutoTuner(sampleFiles = mzml_files, metadata = sample_metadata)
```

## Evaluation signals

- AutoTuner parameter estimates from random sample subsets yield CV < 0.1 (Monte Carlo robustness check)
- mzDiff sensitivity analysis: feature tables built with mzDiff values differing by 0.001 show minimal differences in unique feature counts
- Parameter sweep size drops from ≥24×5^8 to ~40 combinations, confirming 7 of 8 continuous centWave parameters were estimated
- Manual inspection of raw data passes quality control before downstream analysis
- Tuned processing yields smaller feature tables with more information per feature

## Limitations

- Group difference parameter robustness cannot be checked automatically; it requires a non-automatable cross-sample comparison during TIC peak detection
- AutoTuner's divide-and-conquer speed gain sacrifices comparing EIC peaks across samples to estimate retention time correction algorithms
- The sliding window analysis requires user input during the first portion of AutoTuner
- AutoTuner is not parallelized; speed comes at ~100–1000X over IPO, but parallelization remains a possible future improvement
- Paucity of size-validated metabolome datasets precludes further evaluation of the feature-number comparison

## Evidence

- [intro] Parameter estimates are single-step inferences: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [intro] Parameter optimization prevents noise inflation: "parameter optimization must occur prior to analysis to avoid noise inflation within the feature table"
- [discussion] Sensitivity analysis scope for mzDiff: "In regards to the last continuous parameter not optimized by AutoTuner, mzDiff, we performed a sensitivity analysis"
- [discussion] Sample-size recommendation for CV < 0.1: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [readme] mzML input requirement: [evidence span withheld — anchor status: external_doc]
