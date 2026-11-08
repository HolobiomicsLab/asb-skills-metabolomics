---
name: sliding-window-signal-analysis
description: Use when processing raw untargeted metabolomics data (.mzML, .mzXML,
  .mzData, .CDF from qTOF, orbitrap, or FT-ICR instruments) to detect TIC peaks per
  sample and derive dataset-specific peak-picking parameters in a single inference
  step rather than iterative parameter sweeping.
when_to_use_negative:
- Input is already a processed feature table — AutoTuner operates on raw chromatogram
  data to estimate parameters prior to feature-table generation
- The goal is cross-sample EIC peak comparison or retention time correction — AutoTuner's
  divide-and-conquer approach deliberately does not compare EIC peaks across samples
- A fully automated pipeline is required with no user interaction — the sliding window
  analysis requires user input during the first portion of AutoTuner
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: AutoTuner
  role: R package providing the ThresholdingAlgo sliding-window function and linear-model
    boundary refinement for TIC peak detection on raw data
  repo: https://github.com/crmclean/autotuner
- name: R
  role: Runtime environment; AutoTuner requires R >= 3.6 and is installed via BioConductor
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:bc461c26bfdffaaee447abff03d4c2723efea8ecf9f19752ce812c7a878ee992
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/crmclean/autotuner
  tools:
  - AutoTuner
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sliding-window-signal-analysis

## Summary

Applies AutoTuner's ThresholdingAlgo sliding-window analysis over each sample's total ion chromatogram (TIC) to flag local signal deviations as candidate TIC peaks, then refines peak boundaries with an iterative linear-model fit (R2 >= 0.8) on the first and last three bounding points.

## When to use

Use when processing raw untargeted metabolomics data (.mzML, .mzXML, .mzData, .CDF from qTOF, orbitrap, or FT-ICR instruments) to detect TIC peaks per sample and derive dataset-specific peak-picking parameters in a single inference step rather than iterative parameter sweeping.

## When NOT to use

- Input is already a processed feature table — AutoTuner operates on raw chromatogram data to estimate parameters prior to feature-table generation
- The goal is cross-sample EIC peak comparison or retention time correction — AutoTuner's divide-and-conquer approach deliberately does not compare EIC peaks across samples
- A fully automated pipeline is required with no user interaction — the sliding window analysis requires user input during the first portion of AutoTuner

## Inputs

- Raw untargeted metabolomics mass spectrometry files (mzML, mzXML, mzData, or CDF) from at least 3 samples, converted from proprietary instrument formats
- Per-sample total ion chromatogram (TIC) extracted from the raw files
- User-specified sliding-window parameters: lag, threshold, influence

## Outputs

- Per-sample TIC peak boundary tables with start and end scan indices and linear-model R2 fit statistics
- autotuner_tic_peaks.csv containing per-sample rows of accepted TIC peaks
- Candidate peaks used downstream to estimate centWave peak-picking parameters

## How to apply

Load each sample's raw chromatogram (e.g., the mtbls2 mzData example files shipped with the AutoTuner R package) and extract the total ion chromatogram. Run the ThresholdingAlgo sliding window over each TIC using the lag, threshold, and influence parameters to flag local deviations as candidate TIC peaks; note this first portion of AutoTuner requires user input for these settings. For each candidate peak, fit a linear model to the region surrounding the peak using the first and last three points bounding it; if the model's R2 falls below 0.8 or the endpoint is not a local maximum, expand the ending bound by one scan and rerun the model until the criterion is met. Record the accepted start and end scan indices per sample along with the model fit statistics, and write the per-sample TIC peak boundary tables to autotuner_tic_peaks.csv.

## Related tools

- **AutoTuner** (R package providing the ThresholdingAlgo sliding-window function and linear-model boundary refinement for TIC peak detection on raw data) — https://github.com/crmclean/autotuner
- **R** (Runtime environment; AutoTuner requires R >= 3.6 and is installed via BioConductor) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html

## Examples

```
BiocManager::install("Autotuner"); peaks <- Autotuner::ThresholdingAlgo(tic, lag, threshold, influence)
```

## Evaluation signals

- autotuner_tic_peaks.csv exists and contains per-sample rows
- Every retained peak entry carries a linear-model R2 value >= 0.8, as required by the bound-expansion criterion
- Each accepted peak has non-empty start and end scan indices for its sample
- Peak boundaries were refined by expanding the ending bound one scan at a time only while R2 < 0.8 or the endpoint was not a local maximum
- At least 3 raw samples were processed, matching the AutoTuner minimum input requirement

## Limitations

- The sliding window analysis requires user input (lag, threshold, influence) during the first portion of AutoTuner; a more sophisticated peak detection approach could eliminate this
- The divide-and-conquer speed gain comes at a loss of comparing EIC peaks across samples to estimate retention time correction
- AutoTuner is not parallelized, so computation is limited by a single CPU

## Evidence

- [readme] Raw format requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] User input in first stage: "the replacement of the sliding window analysis with a more sophisticated and sensitive peak detection approach may eliminate the need for user input during the first portion of AutoTuner"
- [abstract] Single-step inference on raw data: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [discussion] Not parallelized: "First, AutoTuner could be parallelized to reduce computation time by a factor of the total number of CPUs used."
