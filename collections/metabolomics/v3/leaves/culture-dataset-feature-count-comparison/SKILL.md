---
name: culture-dataset-feature-count-comparison
description: Use when a metabolomics parameter-optimization workflow (e.g. AutoTuner)
  leaves mzDiff unoptimized and you need to justify a fixed mzDiff value by showing
  its effect on the returned feature table is minimal.
when_to_use_negative:
- The input is already a processed feature table rather than raw mzML data — the sweep
  requires re-running peak picking per mzDiff value
- mzDiff is itself being optimized (e.g. by IPO); this skill only performs a sensitivity
  check, not parameter optimization
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Runs the centWave findPeaks/group/retcor pipeline once per mzDiff value to
    produce feature tables for comparison
- name: AutoTuner
  role: Supplies the optimal fixed centWave parameters around which mzDiff is varied
  repo: https://github.com/crmclean/autotuner
- name: R
  role: Environment in which the sweep, XCMS processing, and feature-count comparison
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:ab4b8f6ba38d5fe074082769b44ced31ec5fc410633bd2aa580037dd4bc0bc47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/crmclean/autotuner
  tools:
  - AutoTuner
  - BioConductor
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# culture-dataset-feature-count-comparison

## Summary

A sensitivity-analysis skill that quantifies the downstream impact of mzDiff — the one continuous centWave parameter not optimized by AutoTuner — by sweeping it in 0.001 increments and counting unique features in the resulting XCMS feature tables.

## When to use

Use when a metabolomics parameter-optimization workflow (e.g. AutoTuner) leaves mzDiff unoptimized and you need to justify a fixed mzDiff value by showing its effect on the returned feature table is minimal.

## When NOT to use

- The input is already a processed feature table rather than raw mzML data — the sweep requires re-running peak picking per mzDiff value
- mzDiff is itself being optimized (e.g. by IPO); this skill only performs a sensitivity check, not parameter optimization

## Inputs

- Raw untargeted metabolomics mzML files from a culture dataset
- AutoTuner-derived optimal centWave parameters for the dataset (with mzDiff left as the only free variable)

## Outputs

- One feature table per mzDiff setting (from XCMS findPeaks/group/retcor)
- Sensitivity summary table (analogous to Table S5) of feature counts per mzDiff value

## How to apply

Load the raw culture metabolomics dataset (mzML files) into R and configure XCMS centWave peak detection with AutoTuner-derived optimal parameters, leaving mzDiff as the only variable. Execute the full XCMS pipeline (findPeaks/group/retcor) once per mzDiff value, sweeping mzDiff in increments of 0.001 around the nominal value. For each run, collect the returned feature table and record the number of features and peak characteristics. Compile per-run results into a sensitivity summary analogous to Table S5, comparing the number of unique features between pairs of feature tables generated with mzDiff values differing by 0.001. The rationale is that AutoTuner optimizes seven continuous centWave parameters but not mzDiff, so a sensitivity analysis must demonstrate that distinct mzDiff values have minimal effect before fixing it.

## Related tools

- **XCMS** (Runs the centWave findPeaks/group/retcor pipeline once per mzDiff value to produce feature tables for comparison)
- **AutoTuner** (Supplies the optimal fixed centWave parameters around which mzDiff is varied) — https://github.com/crmclean/autotuner
- **R** (Environment in which the sweep, XCMS processing, and feature-count comparison are executed)

## Examples

```
BiocManager::install("Autotuner")  # then sweep mzDiff around the AutoTuner-derived value in R, running XCMS findPeaks/group/retcor per 0.001 increment
```

## Evaluation signals

- Each XCMS run uses identical parameters except mzDiff, which differs by exactly 0.001 between consecutive runs
- Feature counts across distinct mzDiff values differ only minimally, consistent with the reported finding of 'minimal effect on the returned feature table'
- The sensitivity summary table covers the full sweep and records number of features per run
- Results are reproducible with the AutoTuner-derived optimal parameters for the culture dataset

## Limitations

- The sensitivity result was demonstrated on a culture metabolomics dataset; applicability to other datasets is not established
- Only the number of unique features between feature-table pairs was counted — feature identities and annotations were not compared
- mzDiff is the only continuous centWave parameter not optimized by AutoTuner, so this analysis covers just that residual parameter

## Evidence

- [discussion] mzDiff sensitivity sweep: "In regards to the last continuous parameter not optimized by AutoTuner, mzDiff, we performed a sensitivity analysis to show that distinct values had minimal effect on the returned feature table"
- [discussion] parameter reduction context: "reduces the total combinations of available centWave parameters from at least 24* 58 possible choices of parameters to 40."
- [intro] AutoTuner scope: "AutoTuner is capable of selecting parameters for seven continuously valued parameters required for centWave peak selection algorithm used by both MZmine2 and XCMS, and it determines a key parameter"
- [readme] package installation: [evidence span withheld — anchor status: external_doc]
