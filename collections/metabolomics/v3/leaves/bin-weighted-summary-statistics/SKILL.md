---
name: bin-weighted-summary-statistics
description: Use when a table of per-peak AutoTuner parameter estimates must be reduced
  to one dataset-wide eight-parameter set before returning parameters to the user
  or feeding centWave processing in XCMS/MZmine2.
when_to_use_negative:
- Input is already a processed feature table — AutoTuner performs statistical inference
  on raw data in a single step, not on features
- You need to optimize the mzDiff parameter — it is the one continuous centWave parameter
  AutoTuner does not estimate (only a sensitivity analysis was performed)
- Fewer than 3 raw-data samples are available — AutoTuner requires at least 3 samples
  as input
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3364
tools:
- name: AutoTuner
  role: R package that produces the per-peak parameter estimates and executes the
    aggregation step into the dataset-wide parameter set
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Runtime language for AutoTuner (requires R version 3.6 or greater); aggregation
    is performed in R
- name: XCMS
  role: Downstream consumer — the eight aggregated parameters parameterize the centWave
    peak-picking algorithm used by XCMS
- name: MZmine2
  role: Downstream consumer — the eight aggregated parameters parameterize the centWave
    peak-picking algorithm used by MZmine2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:30afc64daf98bc073aed5588f29fdf22342085638890aa280788cb5047f88c1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - MZmine2
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# bin-weighted-summary-statistics

## Summary

Aggregation rule that reduces a table of per-peak AutoTuner parameter estimates into a single dataset-wide eight-parameter vector (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum/Maximum Peak-width, Group difference) used for centWave peak picking in untargeted metabolomics data processing.

## When to use

Use when a table of per-peak AutoTuner parameter estimates must be reduced to one dataset-wide eight-parameter set before returning parameters to the user or feeding centWave processing in XCMS/MZmine2.

## When NOT to use

- Input is already a processed feature table — AutoTuner performs statistical inference on raw data in a single step, not on features
- You need to optimize the mzDiff parameter — it is the one continuous centWave parameter AutoTuner does not estimate (only a sensitivity analysis was performed)
- Fewer than 3 raw-data samples are available — AutoTuner requires at least 3 samples as input

## Inputs

- Table of per-peak AutoTuner parameter estimates from the upstream peak-detection stage
- Raw untargeted metabolomics samples (.mzML, .mzXML, or .CDF) with a sample-metadata spreadsheet mapping sample names to experimental factors (at least 3 samples; 9–12 recommended)

## Outputs

- Single named vector of eight dataset-wide parameter estimates (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum Peak-width, Maximum Peak-width, Group difference)
- autotuner_parameter_estimates.csv

## How to apply

Load the table of per-peak AutoTuner estimates produced by the upstream peak-detection stage and combine them per-parameter: compute bin-weighted averages across peaks for the ppm and S/N Threshold parameters, and take per-parameter minima for scan count, noise, prefilter level, and minimum peak-width. Take the maximum of the cross-sample group-difference estimates and the average of each sample's maximal peak-width to obtain the remaining two values. Assemble all eight aggregated values (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum Peak-width, Maximum Peak-width, Group difference) into a single named parameter vector and write it to autotuner_parameter_estimates.csv. The rationale is to collapse AutoTuner's single-step statistical-inference estimates on raw data into one conservative parameter set; with 9–12 input samples the resulting estimates typically show CV values below 0.1.

## Related tools

- **AutoTuner** (R package that produces the per-peak parameter estimates and executes the aggregation step into the dataset-wide parameter set) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Runtime language for AutoTuner (requires R version 3.6 or greater); aggregation is performed in R)
- **XCMS** (Downstream consumer — the eight aggregated parameters parameterize the centWave peak-picking algorithm used by XCMS)
- **MZmine2** (Downstream consumer — the eight aggregated parameters parameterize the centWave peak-picking algorithm used by MZmine2)

## Examples

```
BiocManager::install("Autotuner")
```

## Evaluation signals

- Output vector contains exactly eight named parameters: ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum Peak-width, Maximum Peak-width, and Group difference
- ppm and S/N Threshold values in the output equal bin-weighted averages recomputable from the per-peak estimate table
- Scan count, Noise, Prefilter intensity, and Minimum Peak-width equal the per-parameter minima across peaks; Group difference equals the cross-sample maximum; Maximum Peak-width equals the mean of per-sample maxima
- autotuner_parameter_estimates.csv is written and parses to eight numeric values
- With 9–12 input samples, recomputed estimates should show CV values below 0.1 for most parameters

## Limitations

- Robustness of the Group difference parameter estimate could not be checked because it is estimated through a non-automatable cross sample comparison during the TIC peak detection step
- Requires at least 3 samples of raw data converted from proprietary instrument formats (.mzML, .mzXML, .CDF) plus a metadata spreadsheet with sample names and experimental factors; R version 3.6 or greater
- AutoTuner is not parallelized, and the sliding window analysis in the first portion requires user input

## Evidence

- [discussion] For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1.: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [intro] AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm"
- [readme] AutoTuner is used to identify dataset specific parameters to process untargeted metabolomics data.: [evidence span withheld — anchor status: external_doc]
- [discussion] We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison: "We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison"
- [readme] For input, AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF).: [evidence span withheld — anchor status: external_doc]
