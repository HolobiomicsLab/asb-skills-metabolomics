---
name: lc-ms-peak-area-normalization
description: Use when MultiQuant-exported peak areas and RTs from a targeted LC-MS/MS
  sequence show intensity drift and a compound needs IS correction, QC-based drift
  correction, custom bracketing, or weighted bracketing before external-calibration
  quantification.
when_to_use_negative:
- Input is raw vendor spectral data requiring chromatographic peak integration or
  re-integration — QuantyFey only processes already integrated mass spectrometry data
- Operating outside Windows for the standalone version or without Apptainer/R-4.2.x
  setup on Linux/macOS, since QuantyFey is Windows-only compatible
- Quantifying untargeted or discovery MS data without external calibration curves
  and defined injection sequence metadata
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QuantyFey
  role: Shiny application implementing IS correction, QC-based statistical-model drift
    correction, custom/weighted bracketing, interactive and automatic regression-model
    optimization, and external-calibration quantification of the normalized peak areas
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: MultiQuant 3.0.3
  role: Vendor software performing peak integration and manual review of chromatographic
    data before export of peak areas and RTs for normalization
- name: R 4.2.0
  role: Runtime environment (R-portable 4.2.0) in which QuantyFey is implemented and
    executed locally
- name: Shiny 1.10.0
  role: Web framework hosting the interactive drift-correction and quantification
    workflow
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:6f79e64984f9e45e4fe312f3085747e520091538bf99d17b2d2e8431349a7055
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  - Shiny 1.10.0
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# lc-ms-peak-area-normalization

## Summary

Normalizes exported peak areas and retention times from targeted LC-MS/MS sequences for drift using internal-standard correction, QC-proxy-based statistical models (linear, loess, polynomial), or IS/bracketing strategies, converting the normalized areas into concentrations via external calibration regression.

## When to use

Use when MultiQuant-exported peak areas and RTs from a targeted LC-MS/MS sequence show intensity drift and a compound needs IS correction, QC-based drift correction, custom bracketing, or weighted bracketing before external-calibration quantification.

## When NOT to use

- Input is raw vendor spectral data requiring chromatographic peak integration or re-integration — QuantyFey only processes already integrated mass spectrometry data
- Operating outside Windows for the standalone version or without Apptainer/R-4.2.x setup on Linux/macOS, since QuantyFey is Windows-only compatible
- Quantifying untargeted or discovery MS data without external calibration curves and defined injection sequence metadata

## Inputs

- MultiQuant-exported peak areas and retention times (RTs) for targeted LC-MS/MS injections (e.g. porcine plasma sequence, 25 compounds)
- Quantifier and qualifier/IS transition peak-area columns per injection
- Injection-order/sequence metadata identifying calibration standards, QC-proxy samples, and blanks

## Outputs

- Drift-corrected/normalized peak-area table per injection and compound
- IS-corrected concentration table (e.g. is_corrected_tryptophan.csv)
- Calibration regression parameters (linear/quadratic) and fitted drift models
- Mean bias, limits of agreement, Passing-Bablok regression, Pearson and Spearman correlation of concentration vs injection position

## How to apply

Load the vendor-exported peak-area/RT table (MultiQuant 3.0.3 export for a sequence such as the porcine plasma targeted LC-MS/MS run with 25 compounds) into the QuantyFey Shiny app, which works only on already integrated data and provides no integration capabilities. Compute IS ratios by dividing quantifier peak areas by IS transition peak areas, substituting the median IS intensity when IS signal falls below 1% of the sequence median (e.g. blanks without added IS), and apply optional sample-type correction factors. For QC-based correction, select a repeatedly injected non-blank sample, fit linear, loess, or polynomial regression to its signal across injection order, then scale each sample's peak area by the ratio of the fitted trend at that position to the trend at the reference position and normalize by the mean modeled QC-proxy value. For bracketing, assign calibration data to predefined sequence blocks or weight calibration curves by sample position between calibration standards; regression models (linear or quadratic) can be optimized interactively or with the automatic optimization module. Convert normalized areas to concentrations with the external calibration regression for each compound and validate against uncorrected values using mean bias, limits of agreement, Passing-Bablok regression, and Pearson/Spearman correlation of concentration vs injection position.

## Related tools

- **QuantyFey** (Shiny application implementing IS correction, QC-based statistical-model drift correction, custom/weighted bracketing, interactive and automatic regression-model optimization, and external-calibration quantification of the normalized peak areas) — https://github.com/CDLMarkus/QuantyFey/releases/
- **MultiQuant 3.0.3** (Vendor software performing peak integration and manual review of chromatographic data before export of peak areas and RTs for normalization)
- **R 4.2.0** (Runtime environment (R-portable 4.2.0) in which QuantyFey is implemented and executed locally)
- **Shiny 1.10.0** (Web framework hosting the interactive drift-correction and quantification workflow)

## Evaluation signals

- Output file (e.g. is_corrected_tryptophan.csv) contains one concentration per injection and no re-integrated peak areas
- Mean bias and limits of agreement between corrected and uncorrected concentrations confirm drift is reduced, consistent with the paper's finding that both QC-based and IS-based correction significantly reduced drift effects
- Pearson/Spearman correlation of concentration vs injection position decreases after correction (drift signal removed)
- IS-corrected concentrations are consistently lower by approximately a factor of two relative to bracketing methods, matching the reported matrix-enhancement factor
- IS ratios no longer inflate for blanks — IS signals below 1% of the sequence median are replaced by the median before ratioing

## Limitations

- IS correction with an unsuitable analogue (e.g. 4-hydroxy-proline or methionine sulfoxide vs proline/methionine) introduces biases when analyte and IS show differing drift trends
- Custom or weighted bracketing improved accuracy but showed variable performance across compounds; weighted bracketing performed worse than loess-based approaches in simulations
- QuantyFey is compatible with Windows operating systems only (standalone version); Linux/macOS require Apptainer or local R launch with additional prerequisites
- Standalone-version package installation may take up to 10 minutes on first launch
- No changelog found

## Evidence

- [intro] QuantyFey is a Shiny application for the visualization, analysis, and quantification of mass spectrometry (MS) data using external calibration: "QuantyFey is a Shiny application for the visualization, analysis, and quantification of mass spectrometry (MS) data using external calibration"
- [intro] It is specifically designed to address intensity drifts in datasets, offering multiple correction strategies: "It is specifically designed to address **intensity drifts** in datasets, offering multiple **correction strategies** to ensure accurate quantification"
- [abstract] Peak areas and retention times (RTs) were exported: "Peak integration and manual review of chromatographic data were performed in MultiQuant (v3.0.3, Sciex). Peak areas and retention times (RTs) were exported"
- [abstract] Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
- [methods] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration capabilities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [readme] Weighted Bracketing Weighting of calibration curves based on the position of the samples between calibration curves.: "**Weighted Bracketing** Weighting of calibration curves based on the position of the samples between calibration curves."
- [readme] Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards.: "**Automatic Optimization Module** - automatic selection of **linear** or **quadratic** regression model, and selection of appropriate standards."
- [intro] QuantyFey is compatible with Windows operating systems only.: "QuantyFey is compatible with **Windows** operating systems only."
- [abstract] the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [abstract] for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues"
