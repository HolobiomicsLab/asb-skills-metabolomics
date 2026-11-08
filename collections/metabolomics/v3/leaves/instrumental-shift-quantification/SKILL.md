---
name: instrumental-shift-quantification
description: Use when preparing raw LC-MS-based metabolomics data for feature extraction
  and you need an automatically measured instrumental shift value rather than a manually
  guessed retention-time drift tolerance.
when_to_use_negative:
- Input is already a processed feature table or peak list — Paramounter measures parameters
  from raw LC-MS data prior to feature extraction, so it cannot recover a shift from
  post-extraction outputs.
- The downstream feature-extraction program accepts only manual, vendor-given defaults
  and you do not intend to replace them with white-box measured values.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: Paramounter
  role: R program that automatically measures the universal parameters — including
    instrumental shift — from raw LC-MS data and translates them to software-specific
    values
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: Runtime environment in which Paramounter is executed to measure the parameters
- name: MS-DIAL
  role: Downstream feature-extraction program whose default parameters can be replaced
    by Paramounter's measured instrumental-shift and related values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:2d83d13cfeac4348a62a59cf12883f1637eb31aa0156bb4714d05ccb707a8289
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - MS-DIAL
  - Paramounter
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# instrumental-shift-quantification

## Summary

Directly measures the universal instrumental-shift parameter from raw LC-MS metabolomics data using Paramounter, one of the four universal parameters (mass tolerance, peak height, peak width, instrumental shift) it computes in a white-box fashion prior to feature extraction.

## When to use

Use when preparing raw LC-MS-based metabolomics data for feature extraction and you need an automatically measured instrumental shift value rather than a manually guessed retention-time drift tolerance.

## When NOT to use

- Input is already a processed feature table or peak list — Paramounter measures parameters from raw LC-MS data prior to feature extraction, so it cannot recover a shift from post-extraction outputs.
- The downstream feature-extraction program accepts only manual, vendor-given defaults and you do not intend to replace them with white-box measured values.

## Inputs

- raw LC-MS-based metabolomics data files (instrument spectra, e.g. vendor/acquisition-mode-dependent raw files)
- Paramounter R package environment

## Outputs

- structured parameter record containing fields for mass tolerance, peak height, peak width, and instrumental shift
- software-specific translated parameter values ready for feature-extraction programs

## How to apply

Load the Paramounter R package (github.com/HuanLab/Paramounter) and point it at the raw LC-MS data file so it reads the instrument spectra. Run Paramounter's automatic measurement routine, which summarizes the universal parameters — mass tolerance, peak height, peak width, and instrumental shift — directly from the raw data using rank-based intensity sorting and zone-of-interest concepts. The measured instrumental shift is exported into a structured parameter record alongside the other universal parameters. Then translate the universal instrumental-shift value into the software-specific parameter of the downstream feature-extraction program (e.g. MS-DIAL, XCMS CentWave) before running the pipeline. Validate that the output record contains a plausible numeric instrumental-shift field; Paramounter's optimized parameter sets yielded a threefold increase in extracted metabolites versus MS-DIAL defaults and 3.7-/1.6-fold more true positive features than AutoTuner/IPO.

## Related tools

- **Paramounter** (R program that automatically measures the universal parameters — including instrumental shift — from raw LC-MS data and translates them to software-specific values) — https://github.com/HuanLab/Paramounter
- **R** (Runtime environment in which Paramounter is executed to measure the parameters)
- **MS-DIAL** (Downstream feature-extraction program whose default parameters can be replaced by Paramounter's measured instrumental-shift and related values)

## Examples

```
library(Paramounter); measure_params(raw_lcms_file)
```

## Evaluation signals

- The output parameter record contains a field for instrumental shift alongside mass tolerance, peak height, and peak width.
- Each universal-parameter field, including instrumental shift, holds a plausible numeric value for the LC-MS acquisition mode and MS vendor in use.
- Feature extraction with the measured parameters yields more true positive features than the same tool's default settings (article reports threefold gain over MS-DIAL defaults).
- Paramounter's parameter set is benchmarked as 3.7-fold more true positive features than AutoTuner and 1.6-fold more than IPO.
- The measured values were validated across 11 datasets covering different sample types, data acquisition modes, and MS vendors.

## Limitations

- Paramounter's optimized peak height maximizes true positive features but raises the false positive rate and the likelihood of software crash; users may need a higher peak-height threshold (e.g. 2× the optimized value) — this tuning interacts with the instrumental-shift-driven feature window.
- Validated on 11 datasets; robustness outside the tested sample types, acquisition modes, and MS vendors is not demonstrated in the provided context.
- The article's discussion section was not available in the provided context, so discussion-level caveats could not be extracted.

## Evidence
