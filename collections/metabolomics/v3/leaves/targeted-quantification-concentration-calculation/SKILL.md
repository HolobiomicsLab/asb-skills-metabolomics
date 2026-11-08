---
name: targeted-quantification-concentration-calculation
description: Use when a MultiQuant-exported peak-area table with injection order,
  sample-type annotations, and analyte/IS intensities must be back-calculated to concentrations
  under observed intensity drift and no commercial platform offers bracketing or QC-based
  correction.
when_to_use_negative:
- Raw, unintegrated chromatographic data — QuantyFey works only as an add-on to already
  integrated MS data and provides no integration capabilities
- Datasets acquired on macOS via the standalone version — QuantyFey standalone is
  Windows-only (use R/RStudio or Apptainer routes instead)
- When a validated internal standard exists for every compound and no drift is observed
  — IS correction alone may suffice; this skill's bracketing variants matter most
  when IS are unavailable or unsuitable
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QuantyFey
  role: Shiny application that performs external-calibration quantification with IS,
    QC-based, custom-bracketing, and weighted-bracketing drift correction and interactive/automatic
    regression optimization
  repo: https://github.com/CDLMarkus/QuantyFey
- name: MultiQuant
  role: Upstream peak integration and manual chromatographic review whose exported
    peak areas and RTs feed the quantification workflow
- name: R
  role: Runtime (v4.2.0) in which QuantyFey is implemented and run locally
- name: Shiny
  role: Framework (v1.10.0) providing the interactive GUI for regression model optimization
    and strategy selection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:314d1377ffe9dd7a3570acd8cd084dd8a070e0e4015ca60ee7435fa2c78b4285
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CDLMarkus/QuantyFey
  tools:
  - MultiQuant
  - QuantyFey
  - R
  - R 4.2.0
  - Shiny
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# targeted-quantification-concentration-calculation

## Summary

Post-integration quantification of targeted LC-MS/MS analytes by external calibration regression (linear or quadratic, with weight/level selection) combined with drift-correction strategies (IS correction, QC-based correction, custom bracketing, weighted bracketing), as implemented in the QuantyFey Shiny app.

## When to use

Use when a MultiQuant-exported peak-area table with injection order, sample-type annotations, and analyte/IS intensities must be back-calculated to concentrations under observed intensity drift and no commercial platform offers bracketing or QC-based correction.

## When NOT to use

- Raw, unintegrated chromatographic data — QuantyFey works only as an add-on to already integrated MS data and provides no integration capabilities
- Datasets acquired on macOS via the standalone version — QuantyFey standalone is Windows-only (use R/RStudio or Apptainer routes instead)
- When a validated internal standard exists for every compound and no drift is observed — IS correction alone may suffice; this skill's bracketing variants matter most when IS are unavailable or unsuitable

## Inputs

- Exported MultiQuant (v3.0.3, Sciex) peak-area table with peak areas, retention times, injection order, and sample-type annotations
- Per-compound analyte and internal-standard intensity columns
- Calibration-standard and QC injection annotations defining sequence blocks
- Preselected regression parameters (model form, weights, standard levels) for manual optimization

## Outputs

- Back-calculated concentrations for each compound per injection under each drift-correction strategy
- Optimized linear or quadratic calibration regression models per compound
- Method-comparison statistics: mean bias, limits of agreement, Passing-Bablok regression, Pearson and Spearman correlation of concentration vs injection position

## How to apply

Import the exported peak-area table (peak areas and RTs exported from MultiQuant 3.0.3) into QuantyFey (R 4.2.0, Shiny 1.10.0), which operates on already integrated data and provides no integration itself. For each compound choose among up to four strategies: IS correction, QC-based statistical drift correction, custom bracketing (assigning calibration data to predefined sequence blocks), or weighted bracketing (weighting calibration curves by each sample's position between calibration curves; linear weighting by default). Select a linear or quadratic regression model per compound, either by interactive manual optimization of weights, standard levels, and model form, or via the automatic optimization module that selects model and appropriate standards. Compare strategies per compound rather than picking one globally: weighted bracketing with a linear weight change gave results highly similar to non-linear loess QC-based correction, but custom/weighted bracketing showed variable performance across compounds, and unsuitable IS choices (e.g. 4-hydroxy-proline, methionine sulfoxide) introduced biases.

## Related tools

- **QuantyFey** (Shiny application that performs external-calibration quantification with IS, QC-based, custom-bracketing, and weighted-bracketing drift correction and interactive/automatic regression optimization) — https://github.com/CDLMarkus/QuantyFey
- **MultiQuant** (Upstream peak integration and manual chromatographic review whose exported peak areas and RTs feed the quantification workflow)
- **R** (Runtime (v4.2.0) in which QuantyFey is implemented and run locally)
- **Shiny** (Framework (v1.10.0) providing the interactive GUI for regression model optimization and strategy selection)

## Examples

```
Rscript app.r  # from the QuantyFey folder after setwd('path_to_QuantyFey_app.r'), or execute QuantyFey.bat on Windows
```

## Evaluation signals

- Weighted-bracketing results should be highly similar to QC-based (loess) drift-correction results, as reported for the linear-weighting case
- Quantification accuracy (mean bias, limits of agreement vs compared methods) should improve relative to uncorrected bracketing, even if performance varies across compounds
- Drift reduction should be verifiable by Spearman correlation of concentration vs injection position approaching zero after correction
- IS-corrected concentrations running consistently lower (~factor of two) than other methods flags matrix enhancement and warrants cross-checking with another strategy
- Per-compound regression should be a justified linear or quadratic fit with explicitly selected standard levels and weights (manual or automatic optimization)

## Limitations

- Custom and weighted bracketing improved quantification accuracy but demonstrated variable performance across compounds, so strategy choice must be compound-specific
- Unsuitable IS selection (e.g. 4-hydroxy-proline and methionine sulfoxide, where analyte and analogue trends diverged) introduced biases
- The standalone QuantyFey build is Windows-only; the Apptainer setup is complicated and slow on macOS
- First standalone launch requires automatic package installation (up to 10 minutes) and approval to run QuantyFey.bat

## Evidence

- [abstract] Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
- [methods] Weighted Bracketing Weighting of calibration curves based on the position of the samples between calibration curves.: "Weighted Bracketing Weighting of calibration curves based on the position of the samples between calibration curves."
- [abstract] QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change: "QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change"
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [intro] QuantyFey is a Shiny application for the visualization, analysis, and quantification of mass spectrometry (MS) data using external calibration: "QuantyFey is a Shiny application for the **visualization, analysis, and quantification** of **mass spectrometry (MS) data** using **external calibration**"
