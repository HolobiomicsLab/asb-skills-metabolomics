---
name: linear-model-bound-fitting
description: Use when candidate peak boundaries from a sliding-window TIC/chromatogram
  peak detection must be sharpened before downstream parameter estimation, and per-peak
  scan bounds with an explicit fit-quality criterion (R2 >= 0.8) are needed.
when_to_use_negative:
- Input is already a processed feature table — boundary refinement operates on raw
  chromatograms, not feature tables
- The goal is optimizing the non-automatable Group difference parameter or the remaining
  centWave parameters via parameter sweeping — those are separate later stages of
  AutoTuner
- Peak detection quality is sufficient from the sliding-window stage alone and user
  inspection has already accepted the coarse bounds
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: AutoTuner
  role: Implements the sliding-window TIC peak detection plus linear-model bound refinement
    in its first (TIC peak detection) stage
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Language in which the linear-model (lm) fitting and R2 >= 0.8 bound-expansion
    loop is executed; AutoTuner is an R/BioConductor package
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:94ffa04f793d1382bdf30f5cc15dc794c203a3e55e7043778dcdca7346c41245
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# linear-model-bound-fitting

## Summary

Refines the retention-time/scan boundaries of peaks detected by a sliding-window analysis (e.g., AutoTuner's ThresholdingAlgo over a total ion chromatogram) by iteratively fitting a linear model to the first and last three points bounding each candidate peak, expanding the ending bound by one scan until the fit's R2 reaches 0.8 and the bound is a local maximum.

## When to use

Use when candidate peak boundaries from a sliding-window TIC/chromatogram peak detection must be sharpened before downstream parameter estimation, and per-peak scan bounds with an explicit fit-quality criterion (R2 >= 0.8) are needed.

## When NOT to use

- Input is already a processed feature table — boundary refinement operates on raw chromatograms, not feature tables
- The goal is optimizing the non-automatable Group difference parameter or the remaining centWave parameters via parameter sweeping — those are separate later stages of AutoTuner
- Peak detection quality is sufficient from the sliding-window stage alone and user inspection has already accepted the coarse bounds

## Inputs

- Raw untargeted metabolomics files (e.g., .mzML, .mzXML, .CDF, mzData) from at least 3 samples, on qTOF, orbitrap, or FT-ICR instruments
- Per-sample total ion chromatograms (TIC) extracted from the raw files
- Candidate peak regions flagged by the ThresholdingAlgo sliding-window analysis (lag, threshold, influence parameters)

## Outputs

- Per-sample table of accepted peak start and end scan indices (e.g., autotuner_tic_peaks.csv)
- Linear-model fit statistics (R2) for each retained peak boundary

## How to apply

First run the sliding-window analysis (ThresholdingAlgo with lag, threshold, and influence parameters) over each sample's total ion chromatogram extracted from raw mzData/mzML files to flag candidate peaks. For each candidate, take the first and last three points bounding the peak and fit a linear model to that region to check the boundary placement. If the model's R2 is below 0.8, or the endpoint is not a local maximum, expand the ending bound by one scan and refit, repeating until the R2 >= 0.8 criterion is met. Record the accepted peak start and end scan indices per sample together with the linear-model fit statistics. The rationale is that the sliding-window stage over-detects coarse deviations, so local linear fitting provides higher-fidelity bounds that AutoTuner then uses for its subsequent parameter estimation steps.

## Related tools

- **AutoTuner** (Implements the sliding-window TIC peak detection plus linear-model bound refinement in its first (TIC peak detection) stage) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Language in which the linear-model (lm) fitting and R2 >= 0.8 bound-expansion loop is executed; AutoTuner is an R/BioConductor package) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html

## Examples

```
fit <- lm(intensity ~ scan, data = rbind(head(peak_pts, 3), tail(peak_pts, 3))); if (summary(fit)$r.squared < 0.8) { end_scan <- end_scan + 1 }
```

## Evaluation signals

- Every retained peak entry in the output table carries a linear-model R2 value >= 0.8, per the bound-expansion criterion
- Output file (e.g., autotuner_tic_peaks.csv) exists and contains per-sample rows
- Each recorded peak has both a start and an end scan index, with end >= start
- Endpoints of refined peaks correspond to local maxima of the TIC, as required by the acceptance rule
- Number of refined peaks is consistent with (i.e., no greater than) the number of candidate peaks flagged by the sliding-window stage

## Limitations

- The sliding-window analysis (and hence the peaks being refined) requires user input during the first portion of AutoTuner; the authors note a more sophisticated peak detection approach may eliminate this need
- The Group difference parameter estimated around the TIC peak detection step is a non-automatable cross-sample comparison, so its robustness could not be checked
- AutoTuner (including this stage) is not parallelized, limiting computation speed

## Evidence

- [full_text] AutoTuner detects peaks with a sliding window analysis and then refines each peak's bounds by fitting a linear model to the first and last three points bounding the peak: [evidence span withheld — anchor status: not_found]
- [full_text] if the model's R2 is below 0.8 or not a local maximum, the ending bound is expanded by one scan and the model rerun until a criterion is met: [evidence span withheld — anchor status: not_found]
- [full_text] accepting the expansion only when the linear model achieves R2 >= 0.8: [evidence span withheld — anchor status: not_found]
- [readme] For input, AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF).: [evidence span withheld — anchor status: external_doc]
