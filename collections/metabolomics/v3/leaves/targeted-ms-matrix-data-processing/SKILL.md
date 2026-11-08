---
name: targeted-ms-matrix-data-processing
description: Use when targeted LC-MS/MS peak areas show injection-order intensity
  drift and concentrations must be computed via external calibration, comparing correction
  strategies (IS, QC-based, custom bracketing, weighted bracketing).
when_to_use_negative:
- Input is raw, un-integrated chromatographic data — QuantyFey 'works as an additional
  software to already integrated mass spectrometry data, and does not provide any
  integration cabailities'
- Dataset shows no intensity drift and no internal standards unavailable — commercial
  platform IS correction alone may suffice
- Compound's IS shows differing drift trends from the analyte (e.g. 4-hydroxy-proline
  vs proline, methionine sulfoxide vs methionine) — IS correction introduces bias
  and a compound-specific strategy must be chosen instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: QuantyFey
  role: Shiny application that performs external-calibration quantification with four
    integrated drift-correction strategies and interactive/automatic regression optimization
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Implementation language of QuantyFey (v4.2.0, Shiny v1.10.0); used to compute
    pairwise Pearson correlation matrices on the exported concentration results
- name: MultiQuant 3.0.3
  role: Vendor software (Sciex) for peak integration and manual review of chromatographic
    data prior to export into QuantyFey
- name: renv 1.1.4
  role: Manages QuantyFey's R package dependencies for reproducible local runs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:b66eb62dc368698eab19ccfaa176114c3d4cd2a7b1485bf018d2887bcff3d42b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  - renv 1.1.4
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# targeted-ms-matrix-data-processing

## Summary

Processes vendor-integrated targeted LC-MS/MS peak area tables through QuantyFey's drift-correction strategies (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) with external calibration to produce comparable concentration matrices and cross-method agreement metrics.

## When to use

Use when targeted LC-MS/MS peak areas show injection-order intensity drift and concentrations must be computed via external calibration, comparing correction strategies (IS, QC-based, custom bracketing, weighted bracketing).

## When NOT to use

- Input is raw, un-integrated chromatographic data — QuantyFey 'works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities'
- Dataset shows no intensity drift and no internal standards unavailable — commercial platform IS correction alone may suffice
- Compound's IS shows differing drift trends from the analyte (e.g. 4-hydroxy-proline vs proline, methionine sulfoxide vs methionine) — IS correction introduces bias and a compound-specific strategy must be chosen instead

## Inputs

- Vendor-exported peak area and retention time table from targeted LC-MS/MS integration (e.g. MultiQuant 3.0.3)
- External calibration standard concentration table (calibration levels for linear/quadratic regression)
- Sequence metadata with injection positions of samples, QCs, and calibration standards

## Outputs

- Per-method concentration table (IS correction, QC-based drift correction, custom bracketing, weighted bracketing)
- Pairwise Pearson correlation matrix across correction strategies
- Method-agreement metrics: mean bias, limits of agreement (LoA), Passing-Bablok regression

## How to apply

Export peak areas and retention times (RTs) from vendor integration software (e.g. MultiQuant 3.0.3, Sciex) and load the resulting table into QuantyFey, which works only on already-integrated MS data and provides no integration itself. For each compound, apply up to four correction approaches — IS correction, QC-based drift correction, custom bracketing (calibration data assigned to predefined sequence blocks), and weighted bracketing (linear weighting of calibration curves by sample position) — setting drift-correction parameters and optimizing calibration regression models both automatically and interactively. Export the per-method concentration table (e.g. 25 porcine plasma compounds) and evaluate agreement using mean bias (average relative difference between methods), limits of agreement (LoA), Passing-Bablok regression, and Pearson/Spearman correlations of concentration versus injection position. Validate the processing by checking that QC-based, custom, and weighted bracketing concentrations mutually correlate at Pearson r > 0.9, while default bracketing and IS correction correlate lower with other methods; expect IS-corrected results consistently lower by roughly a factor of two, indicating matrix enhancement.

## Related tools

- **QuantyFey** (Shiny application that performs external-calibration quantification with four integrated drift-correction strategies and interactive/automatic regression optimization) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Implementation language of QuantyFey (v4.2.0, Shiny v1.10.0); used to compute pairwise Pearson correlation matrices on the exported concentration results)
- **MultiQuant 3.0.3** (Vendor software (Sciex) for peak integration and manual review of chromatographic data prior to export into QuantyFey)
- **renv 1.1.4** (Manages QuantyFey's R package dependencies for reproducible local runs)

## Examples

```
cor(conc_table[, c("default_bracketing","qc_drift","custom_bracketing","weighted_bracketing")], method='pearson')
```

## Evaluation signals

- Pairwise Pearson r > 0.9 among the three non-IS strategies (QC-based drift correction, custom bracketing, weighted bracketing) across all compounds
- IS-corrected concentrations consistently lower by approximately a factor of two relative to other methods, consistent with matrix enhancement
- Reduced drift effects confirmed by Spearman correlation of concentration versus injection position and by mean bias/LoA against reference concentrations
- QC-based drift correction and weighted bracketing yield highly similar results despite linear weighting versus non-linear loess
- Calibration regression models correctly selected as linear or quadratic with appropriate standards via the automatic optimization module

## Limitations

- Standalone version is Windows-only; the Apptainer version is recommended for Linux and is slow and difficult to set up on macOS
- Unsuitable IS selection (e.g. 4-hydroxy-proline and methionine sulfoxide analogues with differing drift trends from their analytes) introduced biases
- Custom and weighted bracketing improved quantification accuracy but showed variable performance across compounds; weighted bracketing performed worse than loess-based approaches in simulation

## Evidence

- [abstract] Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
- [abstract] IS correction ~2x lower: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods."
- [readme] No integration capabilities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [readme] Designed for intensity drifts: "It is specifically designed to address **intensity drifts** in datasets, offering multiple **correction strategies** to ensure accurate quantification"
- [abstract] MultiQuant export: "Peak integration and manual review of chromatographic data were performed in MultiQuant (v3.0.3, Sciex). Peak areas and retention times (RTs) were exported"
- [abstract] Agreement metrics: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
