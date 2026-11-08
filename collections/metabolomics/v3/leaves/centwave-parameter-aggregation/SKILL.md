---
name: centwave-parameter-aggregation
description: Use when AutoTuner peak detection has produced per-peak parameter estimates
  across raw metabolomics samples (mzML, mzXML, CDF) and they must be reduced to one
  dataset-wide centWave parameter vector.
when_to_use_negative:
- Input is already a dataset-wide parameter vector rather than a per-peak estimate
  table
- You need the Group difference parameter fully automated — it is estimated through
  a non-automatable cross-sample comparison during the TIC peak detection step
- You need mzDiff or retention-time correction tuning — mzDiff is the one continuous
  centWave parameter AutoTuner does not optimize
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: R package whose final stage performs the per-peak-to-dataset-wide parameter
    aggregation described here
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Runtime environment; AutoTuner requires R version 3.6 or greater
- name: XCMS
  role: Downstream consumer — the aggregated parameters tune centWave peak picking
    and XCMS grouping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:734f59af00fcdda917551a9684a7339ddc44c62219ab22dfa05cdc802d1dd897
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# centwave-parameter-aggregation

## Summary

Final integration stage of the AutoTuner R/BioConductor package that collapses the table of per-peak parameter estimates into a single dataset-wide set of eight centWave/XCMS parameters (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum/Maximum Peak-width, Group difference) for untargeted metabolomics data processing.

## When to use

Use when AutoTuner peak detection has produced per-peak parameter estimates across raw metabolomics samples (mzML, mzXML, CDF) and they must be reduced to one dataset-wide centWave parameter vector.

## When NOT to use

- Input is already a dataset-wide parameter vector rather than a per-peak estimate table
- You need the Group difference parameter fully automated — it is estimated through a non-automatable cross-sample comparison during the TIC peak detection step
- You need mzDiff or retention-time correction tuning — mzDiff is the one continuous centWave parameter AutoTuner does not optimize

## Inputs

- Table of per-peak AutoTuner parameter estimates from the upstream peak-detection stage
- Raw untargeted metabolomics samples in .mzML, .mzXML, or .CDF format (at least 3 samples)
- Spreadsheet with one column matching raw sample names and one describing the experimental factor of each sample

## Outputs

- autotuner_parameter_estimates.csv containing eight aggregated values: ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum Peak-width, Maximum Peak-width, Group difference

## How to apply

Load the table of per-peak AutoTuner parameter estimates produced by the upstream peak-detection stage over raw untargeted metabolomics data. Combine the per-peak values using parameter-specific rules: compute bin-weighted averages across peaks for the ppm and S/N Threshold parameters; take cross-peak minima for Scan count, Noise, Prefilter intensity, and Minimum Peak-width; take the maximum of the cross-sample Group difference estimates; and average each sample's maximal peak-width to obtain Maximum Peak-width. Assemble the eight aggregated values into a single named parameter vector — these tune the seven continuous centWave parameters used by MZmine2 and XCMS plus the key XCMS grouping parameter. Write the result to autotuner_parameter_estimates.csv. The rationale is that AutoTuner performs statistical inference on raw data in a single step rather than iteratively sweeping parameter combinations, so this aggregation is what converts peak-level evidence into the one vector returned to the user.

## Related tools

- **AutoTuner** (R package whose final stage performs the per-peak-to-dataset-wide parameter aggregation described here) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Runtime environment; AutoTuner requires R version 3.6 or greater)
- **XCMS** (Downstream consumer — the aggregated parameters tune centWave peak picking and XCMS grouping)

## Examples

```
BiocManager::install("Autotuner")
```

## Evaluation signals

- Output CSV contains exactly eight named parameters: ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum Peak-width, Maximum Peak-width, Group difference
- ppm and S/N Threshold values equal the bin-weighted averages across all per-peak estimates
- Scan count, Noise, Prefilter intensity, and Minimum Peak-width equal the cross-peak minima
- Group difference equals the maximum across cross-sample estimates, and Maximum Peak-width equals the mean of each sample's maximal peak-width
- Downstream feasibility: the resulting vector reduces centWave parameter combinations from at least 24*5^8 to about 40 choices

## Limitations

- Group difference robustness could not be checked because it is estimated through a non-automatable cross-sample comparison
- AutoTuner is not parallelized, and the divide-and-conquer approach does not compare EIC peaks across samples for retention-time correction
- At least 3 samples are required; 9 to 12 samples are recommended for culture and community datasets to reach CV < 0.1 on most parameters

## Evidence

- [intro] seven continuous centWave parameters plus grouping: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS"
- [intro] single-step statistical inference: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [readme] input data requirements: [evidence span withheld — anchor status: external_doc]
- [discussion] Group difference is not automatable: "We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison"
- [discussion] sample size for CV < 0.1: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
