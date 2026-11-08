---
name: calibration-curve-regression-fitting
description: Use when targeted LC-MS/MS peak-area tables (e.g. exported from MultiQuant)
  must be converted to concentrations via external calibration over a defined range
  (e.g. 1-1000 µg/L), especially when drift correction strategies bracket or correct
  the calibration.
when_to_use_negative:
- Input chromatograms are not yet integrated — QuantyFey works only on already integrated
  MS data and provides no peak-integration capabilities.
- Quantification requires internal-standard-independent methods incompatible with
  external calibration, or the user runs a non-Windows system without the Apptainer
  or local-R launch path.
- The goal is de novo statistical drift modelling of raw intensities rather than calibration-based
  quantification — then QC-based loess drift correction may be more appropriate than
  plain regression fitting.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QuantyFey
  role: Shiny application performing calibration regression fitting (linear/quadratic)
    with automatic and interactive optimization of model, weights, and standard levels,
    plus all four drift-correction/bracketing strategies
  repo: https://github.com/CDLMarkus/QuantyFey
- name: R 4.2.0
  role: Runtime environment in which QuantyFey is implemented and executed locally
    (R-portable 4.2.0)
- name: Shiny 1.10.0
  role: Framework providing the interactive UI for regression-model optimization and
    strategy parameter setup
- name: MultiQuant 3.0.3
  role: Vendor software used upstream for peak integration and export of peak areas
    and retention times consumed by the regression fitting
- name: renv 1.1.4
  role: Manages package dependencies for reproducible QuantyFey execution
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:71a47d0efdb364bb33d05586849d257b2d7e64caba021e9f1fa9a28241c7398a
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
  - Shiny 1.10.0
  - renv 1.1.4
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# calibration-curve-regression-fitting

## Summary

Fit linear or quadratic external-calibration regression models per compound in QuantyFey, with automatic or interactive optimization of model type, weighting, and standard-level selection, to back-calculate concentrations from LC-MS/MS peak areas.

## When to use

Use when targeted LC-MS/MS peak-area tables (e.g. exported from MultiQuant) must be converted to concentrations via external calibration over a defined range (e.g. 1-1000 µg/L), especially when drift correction strategies bracket or correct the calibration.

## When NOT to use

- Input chromatograms are not yet integrated — QuantyFey works only on already integrated MS data and provides no peak-integration capabilities.
- Quantification requires internal-standard-independent methods incompatible with external calibration, or the user runs a non-Windows system without the Apptainer or local-R launch path.
- The goal is de novo statistical drift modelling of raw intensities rather than calibration-based quantification — then QC-based loess drift correction may be more appropriate than plain regression fitting.

## Inputs

- Targeted LC-MS/MS peak-area and retention-time table exported from MultiQuant (calibration standards and samples, analyte/IS intensities, injection order)
- Calibration standard levels spanning 1-1000 µg/L per compound (25 porcine plasma amino acids and biogenic amines in the reference dataset)
- Sequence metadata: sample type flags, calibration-block assignment, and sample positions between calibration curves (for bracketing strategies)

## Outputs

- Per-compound calibration regression coefficients (linear or quadratic, with selected weights and standard levels)
- Back-calculated concentrations per compound and per drift-correction strategy in a combined results table (e.g. weighted_bracketing_results.csv)
- Interactive regression diagnostics for manual model/weight/standard-level adjustment

## How to apply

Load the exported MultiQuant (v3.0.3) peak-area and retention-time table into the QuantyFey Shiny app (R 4.2.0) and, per compound, run the Automatic Optimization Module to select a linear or quadratic regression model and appropriate standard levels spanning the calibration range (1-1000 µg/L). Interactively adjust the model, weights, and included standard levels as a manual check where the automatic fit is unsatisfactory. Decide which bracketing or correction context applies (default bracketing, IS-corrected calibration against analogue IS, custom block assignment, or position-weighted calibration functions between calibration blocks), since regression fitting underlies all four strategies. Back-calculate sample concentrations from the fitted regression per compound and export the concentration table for comparison across strategies. Validate by confirming IS-corrected concentrations run about a factor of two lower than other methods (matrix enhancement) and that QC-based (loess, span 0.75) and weighted-bracketing results are highly similar.

## Related tools

- **QuantyFey** (Shiny application performing calibration regression fitting (linear/quadratic) with automatic and interactive optimization of model, weights, and standard levels, plus all four drift-correction/bracketing strategies) — https://github.com/CDLMarkus/QuantyFey
- **R 4.2.0** (Runtime environment in which QuantyFey is implemented and executed locally (R-portable 4.2.0))
- **Shiny 1.10.0** (Framework providing the interactive UI for regression-model optimization and strategy parameter setup)
- **MultiQuant 3.0.3** (Vendor software used upstream for peak integration and export of peak areas and retention times consumed by the regression fitting)
- **renv 1.1.4** (Manages package dependencies for reproducible QuantyFey execution)

## Examples

```
setwd('path_to_QuantyFey_app.r'); source('app.r')  # launch QuantyFey Shiny app to fit linear/quadratic calibration regressions per compound
```

## Evaluation signals

- IS-corrected back-calculated concentrations are consistently lower by roughly a factor of two relative to other strategies, indicating matrix enhancement — a deviation from this pattern flags misapplied calibration.
- Weighted-bracketing quantification is highly similar to QC-based (loess, span 0.75) drift correction despite linear vs non-linear weighting; large divergence suggests incorrect block/weight assignment.
- Automatic Optimization Module selects linear vs quadratic per compound and appropriate standards; manually inspect/adjust the fit for compounds with poor regression behaviour.
- Exported concentration table contains the expected per-compound/per-strategy fields and reproduced concentrations match the published findings for the 25-compound porcine plasma dataset.
- Compounds with unsuitable analogue IS (e.g. 4-hydroxy-proline, methionine sulfoxide vs proline/methionine) show drift-correction-induced bias — check compound-specific strategy choice.

## Limitations

- QuantyFey is compatible with Windows only for the standalone version; Linux requires Apptainer and macOS setup is slow and difficult.
- The app works as additional software on already integrated MS data and does not provide any integration capabilities.
- Custom- and weighted bracketing improved quantification accuracy but demonstrated variable performance across compounds.
- Unsuitable IS selection (e.g. 4-hydroxy-proline and methionine sulfoxide) introduced biases, underscoring compound-specific strategy selection.

## Evidence

- [abstract] Automatic optimization of calibration regression models: "Parameters for the drift correction strategies were set up and all regression models were manually and automatically optimized."
- [readme] Interactive and automatic regression optimization: "Automatic Optimization Module - automatic selection of **linear** or **quadratic** regression model, and selection of appropriate standards."
- [intro] External calibration scope: "**QuantyFey** is a Shiny application for the **visualization, analysis, and quantification** of **mass spectrometry (MS) data** using **external calibration**"
- [methods] No integration capabilities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [abstract] IS-corrected results lower by factor of two: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods. This suggests matrix enhancement"
