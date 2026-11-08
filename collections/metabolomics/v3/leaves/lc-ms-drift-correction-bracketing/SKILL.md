---
name: lc-ms-drift-correction-bracketing
description: Use when targeted LC-MS/MS peak areas (e.g. exported from MultiQuant
  3.0.3) show intensity drift across injection order and internal standards are unavailable
  or unsuitable, requiring QC-based drift correction or calibration bracketing.
when_to_use_negative:
- Raw, unintegrated chromatographic data — QuantyFey works as additional software
  to already integrated MS data and provides no integration capabilities
- Datasets with no observable intensity drift across injection order, where standard
  internal-standard correction suffices
- Non-Windows environments where the standalone version cannot run and Apptainer setup
  is impractical (e.g. macOS, described as slow and difficult)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QuantyFey
  role: Shiny application implementing the drift-correction strategies (IS correction,
    statistical drift correction, custom bracketing, weighted bracketing), interactive
    and automatic regression optimization, and external-calibration quantification
  repo: https://github.com/CDLMarkus/QuantyFey
- name: MultiQuant 3.0.3
  role: Vendor software for peak integration and manual chromatographic review; exports
    the peak areas and retention times that QuantyFey consumes
- name: R 4.2.0
  role: Runtime for QuantyFey (R-portable 4.2.0 for the standalone version; also runnable
    from local R 4.2.x/4.5.x)
  repo: https://github.com/CDLMarkus/QuantyFey
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:ac245d242660de07f09a5ae2144cb2672b80023128a2706597abe1ed532e1078
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CDLMarkus/QuantyFey
  tools:
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# lc-ms-drift-correction-bracketing

## Summary

Quantify targeted LC-MS/MS peak areas against external calibration while correcting intensity drift, using QuantyFey's four strategies: internal-standard (IS) correction, statistical QC-based drift correction, custom bracketing, and weighted bracketing, followed by linear/quadratic regression optimization.

## When to use

Use when targeted LC-MS/MS peak areas (e.g. exported from MultiQuant 3.0.3) show intensity drift across injection order and internal standards are unavailable or unsuitable, requiring QC-based drift correction or calibration bracketing.

## When NOT to use

- Raw, unintegrated chromatographic data — QuantyFey works as additional software to already integrated MS data and provides no integration capabilities
- Datasets with no observable intensity drift across injection order, where standard internal-standard correction suffices
- Non-Windows environments where the standalone version cannot run and Apptainer setup is impractical (e.g. macOS, described as slow and difficult)

## Inputs

- Peak-area table exported from MultiQuant 3.0.3 with injection order, sample type (calibration standard, QC, sample), and compound analyte/IS intensity columns
- Retention times exported from vendor software
- Sequence map identifying calibration-standard injections for defining bracketing blocks

## Outputs

- Per-compound calibration regression coefficients (linear or quadratic) after drift correction
- Back-calculated sample concentrations per correction strategy (IS, QC-based, custom bracketing, weighted bracketing)
- Weighted-bracketing calibration coefficients and quantified concentrations (e.g. weighted_bracketing_results.csv)
- Method-comparison metrics: mean bias, limits of agreement, Passing-Bablok regression, Pearson and Spearman correlations vs injection position

## How to apply

Import the exported peak-area table (peak areas and retention times, injection order, sample type, analyte/IS intensities) into QuantyFey, a Shiny app running on R-portable 4.2.0. For each compound, choose among four strategies: IS correction, statistical drift correction, custom bracketing (assigning calibration data to predefined sequence blocks), or weighted bracketing (weighting calibration curves by the sample's position between calibration curves; weights transition linearly by default, with loess/polynomial options via technical replicates). Interactively or automatically optimize the regression — selecting linear vs quadratic models, weighting schemes, and appropriate standard levels. Compare methods using mean bias of relative differences, limits of agreement, Passing-Bablok regression, and Pearson/Spearman correlation of concentration vs injection position. Note that QC-based drift correction and weighted bracketing yielded highly similar results in the published porcine plasma dataset, despite linear vs non-linear (loess) weighting, and IS correction was ~2x lower for some analytes, suggesting matrix enhancement.

## Related tools

- **QuantyFey** (Shiny application implementing the drift-correction strategies (IS correction, statistical drift correction, custom bracketing, weighted bracketing), interactive and automatic regression optimization, and external-calibration quantification) — https://github.com/CDLMarkus/QuantyFey
- **MultiQuant 3.0.3** (Vendor software for peak integration and manual chromatographic review; exports the peak areas and retention times that QuantyFey consumes)
- **R 4.2.0** (Runtime for QuantyFey (R-portable 4.2.0 for the standalone version; also runnable from local R 4.2.x/4.5.x)) — https://github.com/CDLMarkus/QuantyFey

## Examples

```
setwd("path_to_QuantyFey_app.r"); source("app.r")
```

## Evaluation signals

- Mean bias (average of relative differences between compared methods) is near zero for corrected vs reference concentrations
- Limits of agreement between methods are narrow relative to the concentration range
- Passing-Bablok regression of corrected vs uncorrected concentrations shows proportional agreement without systematic offset
- Spearman correlation of concentration vs injection position approaches zero after correction, indicating drift removal
- Weighted-bracketing results should be highly similar to QC-based (loess) drift-correction results, as reported in the paper

## Limitations

- Custom- and weighted bracketing improved quantification accuracy but showed variable performance across compounds; unsuitable IS pairing (e.g. 4-hydroxy-proline and methionine sulfoxide) introduced biases
- IS-corrected concentrations were consistently lower by approximately a factor of two, suggesting matrix enhancement in the porcine plasma LC-MS/MS dataset
- In a simulation, weighted bracketing performed worse than loess-based drift-correction approaches
- Standalone QuantyFey runs on Windows only; the Apptainer version is complicated and slow on macOS
- Package dependencies are managed with renv 1.1.4 and automatic installation on first launch can take up to 10 minutes

## Evidence

- [methods] Weighted Bracketing Weighting of calibration curves based on the position of the samples between calibration curves.: "Weighted Bracketing Weighting of calibration curves based on the position of the samples between calibration curves."
- [abstract] Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the: "Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the"
- [abstract] Custom- or weighted bracketing methods also improved quantification accuracy but demonstrated variable performance across compounds.: "Custom- or weighted bracketing methods also improved quantification accuracy but demonstrated variable performance across compounds."
- [methods] the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [abstract] QuantyFey is implemented in R (v4.2.0) and utilizes the Shiny framework (v1.10.0): "QuantyFey is implemented in R (v4.2.0) and utilizes the Shiny framework (v1.10.0)"
