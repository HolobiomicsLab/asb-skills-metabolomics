---
name: mzdiff-parameter-sensitivity-analysis
description: Use when a centWave (XCMS/MZmine2) metabolomics workflow leaves mzDiff
  unoptimized and you need to quantify how mzDiff variation affects the downstream
  feature table.
when_to_use_negative:
- The goal is dataset-specific parameter optimization itself — use AutoTuner, which
  selects seven continuous centWave parameters plus the Group difference parameter
  in a single statistical-inference step rather than sweeping
- Input is already a processed feature table rather than raw mzML/mzXML/CDF files
  — centWave peak detection and retcor cannot be re-run on it
- Fewer than 3 raw samples are available — AutoTuner requires at least 3 samples of
  raw data to generate parameter estimates
edam_topics: []
tools:
- name: AutoTuner
  role: Supplies the fixed, dataset-specific optimal centWave parameters around which
    the mzDiff sweep is performed
  repo: https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- name: XCMS
  role: Executes the centWave findPeaks/group/retcor pipeline once per mzDiff value
    to produce the feature tables being compared
- name: R
  role: Environment in which AutoTuner (a BioConductor R package, R >= 3.6) and the
    XCMS sweep are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:bddfadaaf8eccf7a7983a66480d0b950292f785b078a9a4fa893033795b17da6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - BioConductor
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

# mzDiff parameter sensitivity analysis

## Summary

Quantifies the impact of mzDiff — the only continuous centWave parameter not optimized by AutoTuner — by sweeping it in 0.001 increments through the XCMS pipeline and comparing the resulting feature tables.

## When to use

Use when a centWave (XCMS/MZmine2) metabolomics workflow leaves mzDiff unoptimized and you need to quantify how mzDiff variation affects the downstream feature table.

## When NOT to use

- The goal is dataset-specific parameter optimization itself — use AutoTuner, which selects seven continuous centWave parameters plus the Group difference parameter in a single statistical-inference step rather than sweeping
- Input is already a processed feature table rather than raw mzML/mzXML/CDF files — centWave peak detection and retcor cannot be re-run on it
- Fewer than 3 raw samples are available — AutoTuner requires at least 3 samples of raw data to generate parameter estimates

## Inputs

- Raw untargeted metabolomics data converted from proprietary instrument formats to .mzML (also .mzXML or .CDF accepted by AutoTuner)
- AutoTuner-derived optimal centWave parameter estimates for the dataset (all seven optimized continuous parameters fixed; only mzDiff varies)
- A sample metadata spreadsheet mapping sample names to experimental factors

## Outputs

- One feature table per mzDiff setting, with feature counts and peak characteristics per run
- A sensitivity summary table (analogous to Table S5) of unique-feature counts between pairs of mzDiff values differing by 0.001

## How to apply

Load the raw metabolomics mzML files (e.g. the culture dataset) into R and configure XCMS centWave peak detection with the AutoTuner-derived optimal parameters, holding every optimized parameter fixed except mzDiff. Execute the full XCMS processing pipeline (findPeaks/group/retcor) once per mzDiff value, sweeping mzDiff in increments of 0.001 around the nominal value. For each run, record the number of features and peak characteristics in the returned feature table. Compile the per-run results into a sensitivity summary (analogous to Table S5) and count the number of unique features between pairs of feature tables generated with mzDiff values differing by 0.001. The analysis is complete when differences in the returned feature tables across distinct mzDiff values are shown to be minimal, justifying the decision to leave mzDiff unoptimized by AutoTuner.

## Related tools

- **AutoTuner** (Supplies the fixed, dataset-specific optimal centWave parameters around which the mzDiff sweep is performed) — https://bioconductor.org/packages/devel/bioc/html/Autotuner.html
- **XCMS** (Executes the centWave findPeaks/group/retcor pipeline once per mzDiff value to produce the feature tables being compared)
- **R** (Environment in which AutoTuner (a BioConductor R package, R >= 3.6) and the XCMS sweep are run)

## Examples

```
BiocManager::install(version='devel'); BiocManager::install("Autotuner")
```

## Evaluation signals

- All centWave parameters except mzDiff are held at the AutoTuner-derived optimal values across every run of the sweep
- mzDiff is swept in increments of exactly 0.001 around the nominal value, with one complete findPeaks/group/retcor run per value
- Feature count per mzDiff setting is recorded, and unique-feature counts between pairs of mzDiff settings are tabulated (Table S5-style summary)
- The reported outcome is reproduced: distinct mzDiff values have minimal effect on the returned feature table
- Every run consumes the identical set of raw mzML files, so any feature-table differences are attributable to mzDiff alone

## Limitations

- mzDiff is the one continuous parameter not optimized by AutoTuner; this sensitivity analysis is the paper's only treatment of it, so it substitutes for optimization rather than providing an optimal value
- The robustness of the Group difference parameter could not be checked, as it is estimated through a non-automatable cross-sample comparison during the TIC peak detection step
- AutoTuner requires R >= 3.6, at least 3 raw samples, and a sample spreadsheet; data must be converted from proprietary formats to .mzML/.mzXML/.CDF before use

## Evidence

- [discussion] Sensitivity analysis showed distinct mzDiff values had minimal effect on the returned feature table: "In regards to the last continuous parameter not optimized by AutoTuner, mzDiff, we performed a sensitivity analysis to show that distinct values had minimal effect on the returned feature table"
- [discussion] centWave requires tuning of 11 distinct parameters across XCMS and MZmine2: "the centWave algorithm, used by both XCMS and MZmine2 data processing tools, requires tuning of 11 distinct parameters"
- [intro] AutoTuner distributed as an R package through BioConductor: "AutoTuner is freely distributed through BioConductor as an R package."
- [readme] Raw-data format and minimum sample requirements for AutoTuner: [evidence span withheld — anchor status: external_doc]
