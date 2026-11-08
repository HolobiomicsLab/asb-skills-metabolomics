---
name: cross-sample-group-difference-estimation
description: Use when running the TIC peak detection stage of AutoTuner and you must
  supply the Group difference value, the sole parameter estimated through a manual,
  non-automatable cross-sample comparison rather than statistical inference on raw
  data.
when_to_use_negative:
- Estimating any of the seven continuous centWave parameters (ppm, S/N Threshold,
  Scan count, Noise, Prefilter intensity, Minimum/Maximum Peak-width) — these come
  from AutoTuner's single-step statistical inference, not this manual comparison
- You expect a fully automated, statistically validated estimate — this parameter
  is explicitly non-automatable and its robustness could not be checked in the Monte
  Carlo experiment
- Input is already a feature table — AutoTuner operates on raw converted data (.mzML/.mzXML/.CDF)
  prior to feature-table generation
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3279
tools:
- name: AutoTuner
  role: R package whose TIC peak detection step surfaces the per-peak TIC peaks that
    the analyst manually compares across experimental factor groups to estimate Group
    difference
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Runtime environment for AutoTuner (requires R version 3.6 or greater) in which
    the cross-sample comparison and downstream parameter aggregation are performed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:e666037eeb89b7e4ec3985f06b3b7fa699d9bd35a4f9b134d3f827676928b67c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3279
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-sample-group-difference-estimation

## Summary

Estimates the XCMS Group difference parameter—the one non-automatable parameter in AutoTuner—by manually comparing TIC peak detections across samples in different experimental factor groups, yielding the value that is later aggregated (as a cross-sample maximum) into the final eight-parameter centWave/grouping parameter set.

## When to use

Use when running the TIC peak detection stage of AutoTuner and you must supply the Group difference value, the sole parameter estimated through a manual, non-automatable cross-sample comparison rather than statistical inference on raw data.

## When NOT to use

- Estimating any of the seven continuous centWave parameters (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum/Maximum Peak-width) — these come from AutoTuner's single-step statistical inference, not this manual comparison
- You expect a fully automated, statistically validated estimate — this parameter is explicitly non-automatable and its robustness could not be checked in the Monte Carlo experiment
- Input is already a feature table — AutoTuner operates on raw converted data (.mzML/.mzXML/.CDF) prior to feature-table generation

## Inputs

- At least 3 raw untargeted metabolomics samples converted from proprietary instrument formats (.mzML, .mzXML, or .CDF)
- Metadata spreadsheet with one column matching raw sample names and one column describing the experimental factor of each sample
- TIC peak detections produced by the AutoTuner TIC peak detection step

## Outputs

- A Group difference parameter estimate for the dataset
- Contribution to the maximum of cross-sample group-difference values in the final eight-parameter set (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum/Maximum Peak-width, Group difference) written to autotuner_parameter_estimates.csv

## How to apply

During the TIC peak detection step of AutoTuner, compare detected TIC peaks across samples belonging to different experimental factor groups to estimate the Group difference — this is the 'key XCMS grouping parameter' AutoTuner cannot infer automatically, unlike the seven continuous centWave parameters (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum/Maximum Peak-width, mzDiff excluded). Prepare inputs per the README: at least 3 raw samples converted to .mzML/.mzXML/.CDF (tested on qTOF, orbitrap, and FT-ICR analyzers) plus a spreadsheet whose sample-name column matches the raw data files and whose second column describes each sample's experimental factor. Because the divide-and-conquer design deliberately does not compare EIC peaks across samples, the analyst must perform this cross-sample comparison manually; the resulting estimate is then taken as the maximum of cross-sample group-difference values during the final aggregation of the eight parameters. Support the judgment with manual inspection of the raw data, which the authors treat as the quality-control check for data-generation steps preceding the analysis.

## Related tools

- **AutoTuner** (R package whose TIC peak detection step surfaces the per-peak TIC peaks that the analyst manually compares across experimental factor groups to estimate Group difference) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Runtime environment for AutoTuner (requires R version 3.6 or greater) in which the cross-sample comparison and downstream parameter aggregation are performed)

## Evaluation signals

- The Group difference value entered here appears as the maximum of the cross-sample group-difference estimates in the final eight-parameter vector (ppm, S/N Threshold, Scan count, Noise, Prefilter intensity, Minimum/Maximum Peak-width, Group difference)
- The metadata spreadsheet's sample-name column must exactly match the raw data sample file names, and the factor column must partition the >=3 samples into the groups being compared
- Manual inspection of the raw data should confirm the cross-sample TIC peak comparison, since no automated CV-based robustness check (like the Monte Carlo CV < 0.1 criterion used for other parameters) exists for this estimate
- The resulting parameter set should be complete with exactly eight entries, all seven continuous centWave parameters plus the grouping parameter

## Limitations

- This parameter is estimated through a non-automatable cross sample comparison, so its robustness could not be checked in the Monte Carlo experiment (unlike other parameters, which reached CV < 0.1 with 9 samples)
- The estimate requires user input during the first portion of AutoTuner; replacing the sliding window analysis with a more sensitive peak detection approach might eliminate this manual step
- AutoTuner's divide-and-conquer speedup sacrifices cross-sample EIC peak comparison, which is why retention-time correction and this Group difference estimation fall outside the automated inference
- AutoTuner has been tested only on untargeted data from qTOF, orbitrap, and FT-ICR mass analyzers and is not parallelized

## Evidence

- [discussion] Group difference estimated non-automatably: "this parameter is estimated through a non-automatable cross sample comparison during the TIC peak detection step of the algorithm."
- [discussion] Robustness of Group difference unchecked: "We were unable to check the robustness of the Group difference parameter estimate, as this parameter is estimated through a non-automatable cross sample comparison"
- [intro] AutoTuner covers seven centWave parameters plus a key grouping parameter: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS"
- [readme] Input metadata requirements: [evidence span withheld — anchor status: external_doc]
