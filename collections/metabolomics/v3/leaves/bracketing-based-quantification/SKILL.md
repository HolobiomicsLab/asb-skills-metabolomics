---
name: bracketing-based-quantification
description: Use when targeted LC-MS/MS peak areas (e.g. MultiQuant exports) show
  intensity drift over the injection sequence and calibration standards are interspersed
  with samples, especially when internal standards are unavailable or unsuitable.
when_to_use_negative:
- Input is raw, unintegrated chromatographic data — QuantyFey works as additional
  software on already-integrated MS data and provides no integration capabilities
- The dataset shows no intensity drift over the injection sequence, so a single default
  bracketing baseline (no drift correction) suffices
- Compound-specific cases where analyte and analogue trends diverge (e.g. 4-hydroxy-proline,
  methionine sulfoxide) — bracketing strategy choice must be tailored per compound
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QuantyFey
  role: Shiny application providing Custom Bracketing (calibration data assigned to
    predefined sequence blocks) and Weighted Bracketing (curve weights based on sample
    position between calibration curves) for external-calibration quantification
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: MultiQuant 3.0.3
  role: Vendor software used upstream for peak integration and manual chromatographic
    review; peak areas and RTs are exported as QuantyFey's input
- name: R 4.2.0
  role: Runtime for QuantyFey; the app can be launched from local R/RStudio (R 4.2.x
    or 4.5.x) by running the app.r script
  repo: https://github.com/CDLMarkus/QuantyFey
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:5312aa6090bb417518943a6da814ab893d8b3bad6c3a401c46fdee8c1f1ce794
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  - Shiny
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bracketing-based-quantification

## Summary

Quantify targeted LC-MS/MS analytes against calibration curves positioned around sample blocks (custom bracketing) or weighted by sample position between consecutive calibration curves (weighted bracketing), as implemented in the QuantyFey Shiny app to compensate for signal intensity drift.

## When to use

Use when targeted LC-MS/MS peak areas (e.g. MultiQuant exports) show intensity drift over the injection sequence and calibration standards are interspersed with samples, especially when internal standards are unavailable or unsuitable.

## When NOT to use

- Input is raw, unintegrated chromatographic data — QuantyFey works as additional software on already-integrated MS data and provides no integration capabilities
- The dataset shows no intensity drift over the injection sequence, so a single default bracketing baseline (no drift correction) suffices
- Compound-specific cases where analyte and analogue trends diverge (e.g. 4-hydroxy-proline, methionine sulfoxide) — bracketing strategy choice must be tailored per compound

## Inputs

- Targeted LC-MS/MS peak-area and retention-time table exported from MultiQuant 3.0.3
- Injection sequence metadata identifying calibration standard, QC, and sample positions
- Calibration designator per compound (e.g. 1-1000 µg/L standard levels)

## Outputs

- Per-compound concentration values for custom bracketing and weighted bracketing strategies
- Combined results table with per-compound/per-strategy concentration fields
- Calibration regression models (linear or quadratic) with selected standards and weights

## How to apply

Load the MultiQuant-exported peak-area and retention-time table (e.g. 25 compounds spanning a 1-1000 µg/L calibration range) into QuantyFey (R 4.2.0, Shiny 1.10.0). First compute calibration regressions — linear or quadratic, chosen via the Automatic Optimization Module with interactive manual adjustment of model, weights, and standard levels. For custom bracketing, assign calibration data to predefined blocks of the injection sequence so each sample is quantified against the curves bracketing its block. For weighted bracketing, weight each calibration curve according to the sample's position between consecutive calibration curves, applying a linear weight change over the sequence. Export concentrations per compound and strategy into a combined results table; interpret weighted bracketing against the loess-based QC drift-correction (span 0.75) reference, since the two were reported to yield highly similar results.

## Related tools

- **QuantyFey** (Shiny application providing Custom Bracketing (calibration data assigned to predefined sequence blocks) and Weighted Bracketing (curve weights based on sample position between calibration curves) for external-calibration quantification) — https://github.com/CDLMarkus/QuantyFey/releases/
- **MultiQuant 3.0.3** (Vendor software used upstream for peak integration and manual chromatographic review; peak areas and RTs are exported as QuantyFey's input)
- **R 4.2.0** (Runtime for QuantyFey; the app can be launched from local R/RStudio (R 4.2.x or 4.5.x) by running the app.r script) — https://github.com/CDLMarkus/QuantyFey

## Examples

```
Rscript -e 'setwd("path_to_QuantyFey_app.r"); source("app.r")'
```

## Evaluation signals

- Output table exists with expected per-compound/per-strategy fields covering custom and weighted bracketing
- Weighted-bracketing concentrations are highly similar to QC-based (loess span 0.75) drift-correction results, as reported despite the linear vs non-linear weighting difference
- Bracketing-based concentrations show reduced drift relative to the default bracketing baseline, but with variable performance across compounds
- Weighted-bracketing results are expected to perform worse than loess-based approaches in the simulation comparison
- Selected regression model (linear or quadratic) and standard levels match the Automatic Optimization Module's choices, or a justified manual adjustment

## Limitations

- Weighted bracketing performed worse than loess-based drift correction in the article's simulation, so it should not be assumed equivalent in every dataset
- Bracketing methods demonstrated variable performance across compounds; per-compound validation (e.g. mean bias, limits of agreement, Passing-Bablok and correlation metrics) is needed
- QuantyFey's standalone version runs on Windows only (R-portable 4.2.0); Linux users need Apptainer, and macOS setup is difficult and slow

## Evidence

- [abstract] Custom- or weighted bracketing methods also improved quantification accuracy but demonstrated variable performance across compounds.: "Custom- or weighted bracketing methods also improved quantification accuracy but demonstrated variable performance across compounds."
- [abstract] QC-based and weighted bracketing similarity: "QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the"
- [abstract] Weighted bracketing vs loess in simulation: [evidence span withheld — anchor status: not_found]
- [methods] Custom bracketing definition: "Custom Bracketing Quantification (Assigning calibration data to predefined blocks of the sequence)."
- [readme] Weighted bracketing definition: "Weighted Bracketing** Weighting of calibration curves based on the position of the samples between calibration curves."
- [abstract] Four correction approaches applied per compound: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
