---
name: internal-standard-correction
description: Use when a targeted LC-MS/MS peak-area table contains measured analogue
  internal-standard transitions for each compound and intensity drift over the injection
  sequence must be corrected before external-calibration quantification.
when_to_use_negative:
- The input is raw, un-integrated chromatographic data — QuantyFey 'works as an additional
  software to already integrated mass spectrometry data, and does not provide any
  integration cabailities'
- No analogue internal standard was measured for the target compound (e.g. analytes
  whose analogue IS shows differing drift trends like 4-hydroxy-proline or methionine
  sulfoxide) — use QC-based drift correction, custom bracketing, or weighted bracketing
  instead
- The dataset shows no intensity drift and IS-corrected results would shift quantification
  (the published IS-corrected values were consistently lower by roughly a factor of
  two due to matrix enhancement)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: QuantyFey
  role: Shiny application implementing the IS-correction drift strategy, ratio computation,
    blank median-substitution, and calibration-based quantification
  repo: https://github.com/CDLMarkus/QuantyFey
- name: MultiQuant 3.0.3
  role: Upstream vendor software for peak integration and manual chromatographic review
    whose exported peak areas and RTs feed the IS correction
- name: R 4.2.0
  role: Runtime environment in which the QuantyFey IS-correction module executes locally
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:17d7ba197b2c5323f4590613b4feb560c3c288eafd571067fad6a974188eeaec
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# internal-standard-correction

## Summary

Drift-correction strategy in QuantyFey that quantifies each analyte against its analogue internal standard by computing IS ratios (quantifier peak area divided by IS transition peak area), with blank handling that substitutes the median IS intensity when IS signal falls below 1% of the sequence median. It significantly reduced drift effects in the porcine plasma LC-MS/MS dataset where commercial platforms only support IS correction.

## When to use

Use when a targeted LC-MS/MS peak-area table contains measured analogue internal-standard transitions for each compound and intensity drift over the injection sequence must be corrected before external-calibration quantification.

## When NOT to use

- The input is raw, un-integrated chromatographic data — QuantyFey 'works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities'
- No analogue internal standard was measured for the target compound (e.g. analytes whose analogue IS shows differing drift trends like 4-hydroxy-proline or methionine sulfoxide) — use QC-based drift correction, custom bracketing, or weighted bracketing instead
- The dataset shows no intensity drift and IS-corrected results would shift quantification (the published IS-corrected values were consistently lower by roughly a factor of two due to matrix enhancement)

## Inputs

- MultiQuant (v3.0.3)-exported targeted LC-MS/MS peak-area and retention-time table with analyte quantifier and analogue IS transition peak areas per injection
- Sample-type annotation (samples, blanks, calibration standards, QC) to apply IS-correction correction factors
- Calibration regression model (linear or quadratic) per compound from the QuantyFey automatic optimization module

## Outputs

- Per-injection IS-corrected concentration values for each compound (e.g. is_corrected_tryptophan.csv with one concentration per injection)
- IS-corrected calibration and quantification results exportable into a combined per-compound/per-strategy results table

## How to apply

Load the MultiQuant-exported peak-area table into the QuantyFey Shiny app (R 4.2.0, Shiny 1.10.0) and select the Internal Standard (IS) correction strategy. For every injection, compute the IS response ratio as the analyte quantifier peak area divided by its analogue IS transition peak area, applying the optional sample-type correction factors where defined (e.g. distinct factors for samples versus calibration standards). Compute the median IS transition peak intensity across the sequence and, before ratioing, replace any IS signal below 1% of this median (blanks without added IS) with the median value to prevent inflated IS ratios for blanks. Convert the IS-corrected ratios into concentrations via the compound's external calibration regression (linear or quadratic, selected via the automatic optimization module with interactive manual adjustment). Check for compound-specific IS suitability: differing trends between analyte and analogue IS (e.g. 4-hydroxy-proline vs proline, methionine sulfoxide vs methionine) introduced biases in the published dataset, so per-compound review of corrected trends is a required decision point.

## Related tools

- **QuantyFey** (Shiny application implementing the IS-correction drift strategy, ratio computation, blank median-substitution, and calibration-based quantification) — https://github.com/CDLMarkus/QuantyFey
- **MultiQuant 3.0.3** (Upstream vendor software for peak integration and manual chromatographic review whose exported peak areas and RTs feed the IS correction)
- **R 4.2.0** (Runtime environment in which the QuantyFey IS-correction module executes locally)

## Evaluation signals

- The output file exists and contains exactly one IS-corrected concentration per injection for each compound (e.g. is_corrected_tryptophan.csv)
- IS-corrected concentrations are consistently lower — by approximately a factor of two — than results from bracketing or QC-based strategies, consistent with the reported matrix enhancement
- Drift is reduced: the correlation of IS-corrected concentration versus injection position is lower than for the default (uncorrected) bracketing baseline
- Blanks without added IS do not show inflated IS ratios: their IS intensities below 1% of the sequence median were substituted with the median before ratioing
- Per-compound trend check: no sign reversal between analyte and analogue IS drift (the 4-hydroxy-proline/methionine sulfoxide failure mode) in the corrected series

## Limitations

- IS selection is compound-specific: for 4-hydroxy-proline and methionine sulfoxide, differing trends between analytes and their analogues (proline and methionine) introduced biases, with increased values after correction
- IS-corrected results were consistently lower by approximately a factor of two than other strategies, suggesting matrix enhancement — IS correction is not interchangeable with bracketing/QC-based correction on this dataset
- QuantyFey is compatible with Windows operating systems only for the standalone version (apptainer recommended for Linux; slow on macOS), and it does not perform peak integration itself

## Evidence

- [abstract] Both QC-based and IS-based correction significantly reduced drift effects.: "Both QC-based and IS-based correction significantly reduced drift effects."
- [abstract] IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods.: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods."
- [abstract] for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases"
- [abstract] While IS correction is widely supported by commercial platforms, other correction strategies, such as quality control (QC)-based drift correction, or quantification bracketing, are unavailable.: "While IS correction is widely supported by commercial platforms, other correction strategies, such as quality control (QC)-based drift correction, or quantification bracketing, are unavailable."
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [readme] It is specifically designed to address **intensity drifts** in datasets, offering multiple **correction strategies** to ensure accurate quantification.: "It is specifically designed to address **intensity drifts** in datasets, offering multiple **correction strategies** to ensure accurate quantification."
