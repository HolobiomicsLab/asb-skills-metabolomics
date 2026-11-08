---
name: centwave-peak-detection-parameterization
description: Use when starting an untargeted metabolomics workflow from raw MS files
  and the XCMS/MZmine2 centWave parameters (peakwidth, ppm, snthresh, noise, mzdiff,
  etc.) have not yet been tuned for this dataset.
when_to_use_negative:
- Input is already a processed feature table rather than raw mzML/mzXML/CDF data —
  AutoTuner operates on raw instrument files only.
- Data are not untargeted metabolomics LC-MS from qTOF, Orbitrap, or FT-ICR instruments
  — AutoTuner has been validated only on those analyzer types.
- The goal is retention-time correction algorithm selection — AutoTuner's divide-and-conquer
  design does not compare EIC peaks across samples for that purpose.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: 'Primary engine: performs single-step statistical inference on raw data to
    estimate centWave and grouping parameters.'
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: XCMS
  role: Downstream peak-picking/grouping engine whose centWave parameters are set
    by AutoTuner output.
- name: MZmine2
  role: Alternative centWave data-processing tool whose parameters can be set from
    AutoTuner estimates.
- name: IPO
  role: Iterative baseline comparator; AutoTuner supersedes IPO's multi-day parameter
    sweeps.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:80c3698eeb01e6b9de3eaa818857f75bad5a0614b595ead4f4eee8ae334dd70d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - BioConductor
  - IPO
  - MZmine2
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# centwave-peak-detection-parameterization

## Summary

Derives dataset-specific centWave peak-picking and grouping parameters from raw metabolomics data (e.g. mzML from qTOF, Orbitrap or FT-ICR instruments) using AutoTuner, replacing iterative parameter sweeping (IPO-style) with a single-step statistical inference on extracted-ion chromatograms.

## When to use

Use when starting an untargeted metabolomics workflow from raw MS files and the XCMS/MZmine2 centWave parameters (peakwidth, ppm, snthresh, noise, mzdiff, etc.) have not yet been tuned for this dataset.

## When NOT to use

- Input is already a processed feature table rather than raw mzML/mzXML/CDF data — AutoTuner operates on raw instrument files only.
- Data are not untargeted metabolomics LC-MS from qTOF, Orbitrap, or FT-ICR instruments — AutoTuner has been validated only on those analyzer types.
- The goal is retention-time correction algorithm selection — AutoTuner's divide-and-conquer design does not compare EIC peaks across samples for that purpose.

## Inputs

- Raw untargeted metabolomics LC-MS/MS data in .mzML, .mzXML or .CDF format (≥3 samples)
- A sample metadata spreadsheet with ≥2 columns matching sample names and experimental factors
- R ≥ 3.6 environment

## Outputs

- Estimated centWave continuous parameters (ppm, peakwidth, snthresh, noise, prefilter) for XCMS/MZmine2
- Grouping parameter (band-width) estimate
- Per-sample parameter stability (CV) statistics from a Monte Carlo subset experiment
- Optionally, a sensitivity table quantifying how feature counts vary across mzDiff values

## How to apply

Convert instrument files to .mzML/.mzXML/.CDF and supply at least 3 raw samples plus a sample-metadata sheet whose rows match sample names and carry the experimental factor. AutoTuner performs a TIC-based sliding-window peak detection, then estimates seven continuous centWave parameters (ppm, peakwidth min/max, snthresh, noise, prefilter, mzdiff is excluded) and the band-width grouping parameter in one pass. Set the sample-count Monte Carlo sweep to 9–12 samples (9 for culture, 12 for community data) so parameter CVs fall below 0.1. mzdiff is the one centWave parameter not optimized; test sensitivity by re-running the XCMS pipeline (findPeaks/group/retcor) across mzDiff increments of 0.001 and comparing feature tables — the article found distinct values had minimal effect on the returned feature set.

## Related tools

- **AutoTuner** (Primary engine: performs single-step statistical inference on raw data to estimate centWave and grouping parameters.) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **XCMS** (Downstream peak-picking/grouping engine whose centWave parameters are set by AutoTuner output.)
- **MZmine2** (Alternative centWave data-processing tool whose parameters can be set from AutoTuner estimates.)
- **IPO** (Iterative baseline comparator; AutoTuner supersedes IPO's multi-day parameter sweeps.)

## Examples

```
BiocManager::install(version='devel'); BiocManager::install("Autotuner")
```

## Evaluation signals

- Parameter CV across Monte Carlo subsets < 0.1 for all seven estimated centWave parameters when ≥9 samples are used.
- AutoTuner-derived peakwidth values must bound the peak widths of authentic standards (IPO failed here because standard peak-width fell below IPO's Minimum Peak-width).
- Running the XCMS pipeline twice with mzDiff differing by 0.001 should yield near-identical feature counts (feature-table invariance).
- Total parameter search space collapses from ≥24·5⁸ combinations to ~40 candidate settings.
- Manual QC: inspect the raw TIC chromatograms and AutoTuner's detected peaks to confirm the sliding-window picks align with real chromatographic features.

## Limitations

- Group difference parameter robustness cannot be checked because it requires a non-automatable cross-sample comparison during TIC peak detection.
- AutoTuner is not parallelized; computation time scales serially with sample number.
- The divide-and-conquer design does not compare EIC peaks across samples, so retention-time correction parameters are not estimated.
- The sliding window analysis still requires user input during the first portion of the run.
- mzDiff is not optimized — only a sensitivity analysis supports the default choice.
- Requires R ≥ 3.6; tested only on qTOF, Orbitrap and FT-ICR mass analyzers.

## Evidence

- [intro] AutoTuner performs single-step inference to estimate centWave parameters: "The method performs statistical inference on raw data in a single step in order to make parameter estimates rather than iteratively checking parameter values."
- [intro] AutoTuner selects seven continuous centWave parameters plus grouping parameter: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS"
- [abstract] Monte Carlo robustness with similar parameter estimates: "We performed a Monte Carlo experiment to test the robustness of AutoTuner parameter selection, and found that AutoTuner generated similar parameter estimates from random subsets of samples."
- [readme] Input requirements from README: [evidence span withheld — anchor status: external_doc]
- [discussion] Parameter CV threshold: "For most of the parameters estimated here, 9 samples proved sufficient to obtain estimates with CV values less than 0.1."
- [discussion] mzDiff sensitivity finding: "we performed a sensitivity analysis to show that distinct values had minimal effect on the returned feature table"
- [discussion] IPO peak-width failure mode: "the peak-width of the missing standards was below the Minimum Peak-width parameter selected by IPO"
- [discussion] Recommendation of sample count: "we recommend the use of 9 to 12 samples to generate estimates in culture and community datasets, respectively."
- [discussion] Parameter space reduction: "reduces the total combinations of available centWave parameters from at least 24* 58 possible choices of parameters to 40."
- [discussion] Non-automatable group difference: "this parameter is estimated through a non-automatable cross sample comparison during the TIC peak detection step of the algorithm."
- [discussion] Not parallelized: "First, AutoTuner could be parallelized to reduce computation time by a factor of the total number of CPUs used."
- [discussion] No EIC cross-sample comparison: "The speed gained in the computation through AutoTuner’s “divide-and-conquer” approach comes at a loss of comparing EIC peaks across samples to estimate retention time correction algorithms."
- [discussion] Requires user input in first portion: "may eliminate the need for user input during the first portion of AutoTuner"
- [discussion] Manual inspection recommended: "We support manual inspection of the raw data, because it provides a quality control check for the data generation steps leading up to the analysis."
- [readme] AutoTuner available through BioConductor: [evidence span withheld — anchor status: external_doc]
- [readme] Requires R 3.6 or greater: [evidence span withheld — anchor status: external_doc]
- [readme] Tested on qTOF, orbitrap and FT-ICR: [evidence span withheld — anchor status: external_doc]
- [abstract] Speed improvement over other algorithms: "it is scalable, highly robust, and very fast (∼100-1000X speed improvement from other algorithms going from days to minutes)."
