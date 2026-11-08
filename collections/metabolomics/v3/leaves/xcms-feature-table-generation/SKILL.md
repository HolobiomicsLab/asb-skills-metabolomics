---
name: xcms-feature-table-generation
description: Use when raw untargeted metabolomics data (.mzML/.mzXML/.CDF) from qTOF,
  Orbitrap, or FT-ICR instruments must be converted into a features table via XCMS
  centWave peak detection, grouping, and retention time correction.
when_to_use_negative:
- Input is already a processed features table — this skill operates on raw instrument
  files (.mzML/.mzXML/.CDF) and starts at peak detection
- The immediate goal is higher-confidence feature annotation — the article positions
  this processing step as complementing, not performing, annotation tools
- Fewer than 3 raw samples are available for AutoTuner-based parameter estimation
  — the README specifies at least 3 samples of raw data as input
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: XCMS
  role: Executes the centWave peak detection, grouping, and retention time correction
    pipeline (findPeaks/group/retcor) that generates the features table
- name: AutoTuner
  role: Estimates the seven continuous centWave parameters plus the Group difference
    parameter from raw data in a single step, prior to XCMS processing
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: R
  role: Runtime environment (>= 3.6) in which the XCMS pipeline and the AutoTuner
    BioConductor package are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:956c80af2b0a7f811a43f70cb0adee7989ee107d27a90cbfbbd5410e94354f15
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - BioConductor
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# xcms-feature-table-generation

## Summary

Runs raw untargeted metabolomics files (.mzML) through the XCMS centWave pipeline — peak picking, grouping, and retention time correction (findPeaks/group/retcor) — to produce a features table, using AutoTuner-derived dataset-specific parameters so that optimization precedes analysis.

## When to use

Use when raw untargeted metabolomics data (.mzML/.mzXML/.CDF) from qTOF, Orbitrap, or FT-ICR instruments must be converted into a features table via XCMS centWave peak detection, grouping, and retention time correction.

## When NOT to use

- Input is already a processed features table — this skill operates on raw instrument files (.mzML/.mzXML/.CDF) and starts at peak detection
- The immediate goal is higher-confidence feature annotation — the article positions this processing step as complementing, not performing, annotation tools
- Fewer than 3 raw samples are available for AutoTuner-based parameter estimation — the README specifies at least 3 samples of raw data as input

## Inputs

- Raw untargeted metabolomics data files (.mzML, .mzXML, or .CDF; at least 3 samples)
- Sample metadata spreadsheet with two columns: a sample-name column matching the raw files and an experimental-factor column
- AutoTuner-derived optimal centWave parameters (seven continuous parameters plus the Group difference grouping parameter)
- mzDiff sweep grid in 0.001 increments for the sensitivity variant

## Outputs

- Features table of m/z–retention time features with per-sample intensities (post findPeaks/group/retcor)
- Per-run feature counts and peak characteristics for each mzDiff setting
- Sensitivity summary table analogous to Table S5

## How to apply

Load the raw culture (or community) metabolomics mzML files into R and first estimate dataset-specific parameters with AutoTuner, which selects the seven continuous centWave parameters plus the Group difference parameter from raw data in a single step (at least 3 samples required; 9-12 samples recommended so most estimates have CV values below 0.1). Configure XCMS centWave peak detection with the AutoTuner-derived optima and execute the pipeline (findPeaks/group/retcor) once per parameter setting. Treat mzDiff — the only continuous centWave parameter AutoTuner does not optimize — as the single variable: sweep it in increments of 0.001 around the nominal value, collecting the returned feature table and feature count per run. Compile the per-run results into a sensitivity summary analogous to Table S5, counting unique features between pairs of feature tables to quantify mzDiff impact. The rationale is that parameter optimization must occur prior to analysis to avoid noise inflation within the feature table; the article reports that distinct mzDiff values had minimal effect on the returned feature table.

## Related tools

- **XCMS** (Executes the centWave peak detection, grouping, and retention time correction pipeline (findPeaks/group/retcor) that generates the features table)
- **AutoTuner** (Estimates the seven continuous centWave parameters plus the Group difference parameter from raw data in a single step, prior to XCMS processing) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **R** (Runtime environment (>= 3.6) in which the XCMS pipeline and the AutoTuner BioConductor package are executed)

## Examples

```
xset <- xcmsSet(mzml_files, method="centWave", ppm=params$ppm, snthresh=params$snthresh, mzdiff=params$mzdiff); xset <- group(xset); xset <- retcor(xset)
```

## Evaluation signals

- Feature counts are recorded for every run; distinct mzDiff values swept in 0.001 increments produce minimal differences in the returned feature tables, consistent with the reported sensitivity finding
- Only mzDiff varies across runs — all other centWave parameters remain fixed at AutoTuner-derived optima
- Most AutoTuner parameter estimates from random sample subsets yield CV values below 0.1 when 9-12 samples are used
- Every raw sample maps to a column in the final features table after grouping and retention time correction
- The compiled sensitivity summary mirrors Table S5, quantifying unique-feature counts between pairs of feature tables

## Limitations

- The Group difference parameter is estimated through a non-automatable cross-sample comparison, so the robustness of that estimate could not be checked
- centWave requires tuning of 11 distinct parameters; AutoTuner optimizes only seven continuous ones plus Group difference, and mzDiff is evaluated solely by sensitivity analysis
- AutoTuner's divide-and-conquer speed gain comes at the loss of comparing EIC peaks across samples to estimate retention time correction algorithms
- The sliding window analysis requires user input during the first portion of AutoTuner, and AutoTuner is not parallelized
- AutoTuner requires R >= 3.6 and at least 3 raw samples; it has been tested on qTOF, Orbitrap, and Fourier transform ion cyclotron resonance mass analyzer data

## Evidence

- [abstract] raw data must be processed to remove noise and to align mass-spectral peaks across samples.: "raw data must be processed to remove noise and to align mass-spectral peaks across samples."
- [intro] raw data from untargeted metabolomics experiments must be processed to generate a features table: "raw data from untargeted metabolomics experiments must be processed to generate a features table"
- [intro] parameter optimization must occur prior to analysis to avoid noise inflation within the feature table: "parameter optimization must occur prior to analysis to avoid noise inflation within the feature table"
- [discussion] we performed a sensitivity analysis to show that distinct values had minimal effect on the returned feature table: "we performed a sensitivity analysis to show that distinct values had minimal effect on the returned feature table"
- [discussion] the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters: "the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters"
- [readme] For input, AutoTuner requires at least 3 samples of raw data converted from proprietary instrument formats (eg .mzML, .mzXML, or .CDF).: [evidence span withheld — anchor status: external_doc]
