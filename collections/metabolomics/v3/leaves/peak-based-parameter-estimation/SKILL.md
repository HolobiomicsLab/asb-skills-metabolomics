---
name: peak-based-parameter-estimation
description: Use when starting from raw untargeted metabolomics data (.mzML, .mzXML,
  or .CDF from qTOF, orbitrap, or FT-ICR instruments) and needing dataset-specific
  centWave peak-picking and XCMS grouping parameters before feature-table generation.
when_to_use_negative:
- Input is already a processed feature table — parameters must be estimated from raw
  data prior to peak picking, not retrofitted
- You need retention-time correction parameter estimation — the divide-and-conquer
  approach does not compare EIC peaks across samples for that purpose
- Full automation is required with no user interaction — the sliding window analysis
  in the first portion of AutoTuner requires user input, and the Group difference
  estimate is non-automatable
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoTuner
  role: 'Primary tool: performs single-step statistical inference on raw TIC peaks
    to estimate centWave and grouping parameters'
  repo: https://github.com/crmclean/Autotuner
- name: R
  role: Runtime environment; AutoTuner is an R package (requires R >= 3.6) distributed
    through BioConductor
- name: XCMS
  role: 'Downstream consumer: the estimated parameters tune centWave peak picking
    and a key XCMS grouping parameter'
- name: MZmine2
  role: 'Downstream consumer: centWave parameters estimated here also apply to MZmine2
    data processing'
- name: BioConductor
  role: 'Distribution channel: released and devel versions of the AutoTuner package
    install via BiocManager'
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:d7f75ed2e385a25e4af5948ab69b6389d5f6a7a277f511091fc8e9ba0838cd70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/crmclean/Autotuner
  tools:
  - AutoTuner
  - BioConductor
  - MZmine2
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# peak-based-parameter-estimation

## Summary

Estimate dataset-specific peak-picking parameters for untargeted metabolomics directly from raw data by detecting TIC peaks and statistically aggregating per-peak estimates into the eight centWave/grouping parameters used by XCMS and MZmine2, replacing slow iterative parameter sweeping.

## When to use

Use when starting from raw untargeted metabolomics data (.mzML, .mzXML, or .CDF from qTOF, orbitrap, or FT-ICR instruments) and needing dataset-specific centWave peak-picking and XCMS grouping parameters before feature-table generation.

## When NOT to use

- Input is already a processed feature table — parameters must be estimated from raw data prior to peak picking, not retrofitted
- You need retention-time correction parameter estimation — the divide-and-conquer approach does not compare EIC peaks across samples for that purpose
- Full automation is required with no user interaction — the sliding window analysis in the first portion of AutoTuner requires user input, and the Group difference estimate is non-automatable

## Inputs

- At least 3 samples of raw metabolomics data converted from proprietary instrument formats (.mzML, .mzXML, or .CDF) from qTOF, orbitrap, or FT-ICR mass analyzers
- A spreadsheet (e.g. CSV) with at least two columns: one matching raw-data sample names and one describing the experimental factor each sample belongs to
- R version 3.6 or greater (BioConductor environment)

## Outputs

- A named vector of eight dataset-wide parameter estimates (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum Peak-width, Maximum Peak-width, Group difference)
- autotuner_parameter_estimates.csv containing the aggregated parameter set for downstream XCMS/MZmine2 peak picking

## How to apply

Provide at least 3 raw samples converted to open formats (.mzML, .mzXML, .CDF) plus a sample-metadata spreadsheet with columns matching sample names and describing experimental factors. AutoTuner performs a sliding-window TIC peak detection step (which includes a non-automatable cross-sample comparison to estimate the Group difference parameter) and then uses statistical inference on the raw data in a single step to estimate seven continuous centWave parameters plus a key XCMS grouping parameter. In the final integration stage, per-peak values are combined: bin-weighted averages for ppm and S/N Threshold, minima for Scan count, Noise, Prefilter intensity, and Minimum Peak-width, the maximum Group difference, and the average of each sample's maximal peak-width; the eight values are assembled into a named parameter vector written to autotuner_parameter_estimates.csv. The rationale is that parameter optimization must occur prior to analysis to avoid noise inflation within the feature table, and that single-step inference reduces the ~24*5^8 possible centWave parameter combinations to about 40. Because this is a divide-and-conquer approach, verify outputs against manual inspection of raw data, which the authors support as a quality-control check.

## Related tools

- **AutoTuner** (Primary tool: performs single-step statistical inference on raw TIC peaks to estimate centWave and grouping parameters) — https://github.com/crmclean/Autotuner
- **R** (Runtime environment; AutoTuner is an R package (requires R >= 3.6) distributed through BioConductor)
- **XCMS** (Downstream consumer: the estimated parameters tune centWave peak picking and a key XCMS grouping parameter)
- **MZmine2** (Downstream consumer: centWave parameters estimated here also apply to MZmine2 data processing)
- **BioConductor** (Distribution channel: released and devel versions of the AutoTuner package install via BiocManager) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html

## Examples

```
BiocManager::install("Autotuner"); library(Autotuner)
```

## Evaluation signals

- Output contains exactly eight named parameter values covering ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum Peak-width, Maximum Peak-width, and Group difference
- Monte Carlo robustness check: re-running parameter estimation on random subsets of samples yields similar estimates (the paper reports CV values less than 0.1 for most parameters with 9 samples)
- Reduced parameter search space: combined estimates should collapse ~24*5^8 possible centWave combinations to roughly 40 candidate settings
- Downstream sanity check: XCMS/MZmine2 processing with the returned parameters produces a feature table without noise inflation and, ideally, fewer features with more information per feature
- Manual inspection of the raw data as a quality control check for the data-generation steps leading up to the analysis

## Limitations

- The Group difference parameter is estimated through a non-automatable cross-sample comparison, so its robustness could not be checked
- The sliding window analysis in the first portion of AutoTuner requires user input
- AutoTuner is not parallelized; computation could be reduced by a factor of total CPUs used but is currently single-threaded
- The divide-and-conquer speed gain comes at a loss of comparing EIC peaks across samples to estimate retention time correction
- mzDiff, the one continuous centWave parameter not optimized, is only covered by a sensitivity analysis showing minimal effect on the returned feature table
- Sample-size sensitivity: 9 samples recommended for culture datasets and 12 for community datasets to obtain CV < 0.1

## Evidence

- [intro] The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values.: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [intro] No universal set of parameters exists for all datasets, hence parameter optimization must occur prior to analysis to avoid noise inflation within the feature table.: "No universal set of parameters exists for all datasets, hence parameter optimization must occur prior to analysis to avoid noise inflation within the feature table."
- [full_text] The integration stage combines per-peak values using bin-weighted averages for ppm and S/N Threshold, minima for Scan count/Noise/Prefilter/Minimum Peak-width, the maximum Group difference: [evidence span withheld — anchor status: not_found]
- [readme] For input, AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF).: [evidence span withheld — anchor status: external_doc]
- [discussion] For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1.: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
