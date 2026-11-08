---
name: external-calibration-concentration-calculation
description: Use when you have a vendor-exported peak-area table (e.g. a MultiQuant
  3.0.3 export of peak areas and retention times) for a targeted LC-MS/MS sequence
  and need per-injection concentrations calibrated against external calibration standards,
  optionally with drift correction.
when_to_use_negative:
- Input is raw chromatographic data that still needs peak integration — QuantyFey
  'does not provide any integration cabailities' and only works on already-integrated
  MS data
- Data are already quantified concentrations and only downstream statistics are needed
- A drift-correction method is required that QuantyFey does not implement (e.g. proprietary
  vendor-autoquant correction)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QuantyFey
  role: Shiny application that performs external-calibration quantification and integrated
    drift correction (IS correction, QC-based, custom bracketing, weighted bracketing)
    on integrated MS data
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runtime in which QuantyFey is implemented (R-portable 4.2.0; launch app.r
    from R 4.2.x or 4.5.x)
  repo: https://github.com/CDLMarkus/QuantyFey
- name: MultiQuant 3.0.3
  role: Upstream vendor software used for peak integration and manual chromatographic
    review; its peak-area/RT export is the direct input to quantification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:f52f4868e3603be5ad791ecf1171708768fc28e853408767c86e3c73e4a31151
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# external-calibration-concentration-calculation

## Summary

Compute analyte concentrations from already-integrated targeted LC-MS/MS peak areas via external calibration regression, as implemented in QuantyFey (R 4.2.0/Shiny). Drift-correction strategies (IS correction, QC-based, custom/weighted bracketing) and linear/quadratic regression optimization are integrated so concentrations reflect injection-order intensity drift.

## When to use

Use when you have a vendor-exported peak-area table (e.g. a MultiQuant 3.0.3 export of peak areas and retention times) for a targeted LC-MS/MS sequence and need per-injection concentrations calibrated against external calibration standards, optionally with drift correction.

## When NOT to use

- Input is raw chromatographic data that still needs peak integration — QuantyFey 'does not provide any integration cabailities' and only works on already-integrated MS data
- Data are already quantified concentrations and only downstream statistics are needed
- A drift-correction method is required that QuantyFey does not implement (e.g. proprietary vendor-autoquant correction)

## Inputs

- Vendor-exported peak-area table with quantifier peak areas and retention times (MultiQuant 3.0.3 export)
- IS transition peak areas paired with analyte quantifier peaks (for IS correction)
- Calibration standard levels and sample-type assignments across the injection sequence
- Drift-correction strategy settings (e.g. sample-type correction factors, bracketing blocks, weighting scheme)

## Outputs

- Per-injection concentration table (e.g. is_corrected_tryptophan.csv) with one concentration per injection per compound
- Optimized calibration regression models (linear or quadratic, with weights and selected standard levels)
- Method-comparison statistics (mean bias, limits of agreement, Passing-Bablok/Pearson/Spearman correlations vs injection position)

## How to apply

Load the exported peak-area table (from MultiQuant 3.0.3 integration of quantifier transitions and, for IS correction, IS transition peak areas) into QuantyFey. Assign calibration standards to their levels and optimize the calibration regression — either interactively (manual adjustment of model, weights, and standard levels) or via the Automatic Optimization Module, which selects between linear and quadratic regression and picks appropriate standards. Apply the chosen drift-correction strategy per compound — IS correction (quantifier area divided by IS transition area, with optional sample-type correction factors and substitution of the median IS intensity for IS signals below 1% of the sequence median, e.g. blanks without added IS), QC-based statistical drift correction, custom bracketing, or weighted bracketing based on sample position between calibration curves. Convert drift-corrected response ratios to concentrations via the regression and write per-injection results (e.g. is_corrected_tryptophan.csv). Compare strategies per compound, since bracketing performance is compound-dependent and unsuitable IS analogues (e.g. proline for 4-hydroxy-proline) can introduce bias.

## Related tools

- **QuantyFey** (Shiny application that performs external-calibration quantification and integrated drift correction (IS correction, QC-based, custom bracketing, weighted bracketing) on integrated MS data) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runtime in which QuantyFey is implemented (R-portable 4.2.0; launch app.r from R 4.2.x or 4.5.x)) — https://github.com/CDLMarkus/QuantyFey
- **MultiQuant 3.0.3** (Upstream vendor software used for peak integration and manual chromatographic review; its peak-area/RT export is the direct input to quantification)

## Examples

```
setwd("path_to_QuantyFey_app.r"); source("app.r")  # launch QuantyFey from local R/RStudio per README, then load MultiQuant peak areas in the Shiny UI
```

## Evaluation signals

- Output CSV exists and contains exactly one concentration value per injection for each compound
- Regression model choice is justified: automatic module reports whether linear or quadratic was selected and which standards were used; results are reproducible when re-run with the same settings
- IS-corrected concentrations are consistently lower than bracketing/uncorrected results (approximately a factor of two in the porcine plasma dataset), consistent with reported matrix enhancement
- Concentration-vs-injection-position trends (Spearman/Pearson correlation, mean bias, limits of agreement) show reduced drift relative to uncorrected quantification
- Compounds with unsuitable IS analogues (e.g. 4-hydroxy-proline, methionine sulfoxide) are flagged, since differing analyte/IS trends introduce bias

## Limitations

- QuantyFey is compatible with Windows only for the standalone version; Linux requires Apptainer and macOS setup is slow and difficult
- No peak integration capability — relies on prior integration and manual review in vendor software (MultiQuant 3.0.3)
- IS correction quality depends on compound-specific IS suitability; mismatched analogues introduced biases for 4-hydroxy-proline and methionine sulfoxide
- Custom and weighted bracketing improved accuracy but showed variable performance across compounds; weighted bracketing performed worse than loess-based approaches in simulation
- No changelog found in the repository

## Evidence

- [intro] QuantyFey is a Shiny application for visualization, analysis, and quantification of mass spectrometry (MS) data using external calibration: "**QuantyFey** is a Shiny application for the **visualization, analysis, and quantification** of **mass spectrometry (MS) data** using **external calibration**"
- [abstract] Peak integration and manual review in MultiQuant precede quantification: "Peak integration and manual review of chromatographic data were performed in MultiQuant (v3.0.3, Sciex). Peak areas and retention times (RTs) were exported"
- [readme] QuantyFey works only on already-integrated MS data: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [abstract] Regression models are optimized manually and automatically: "Parameters for the drift correction strategies were set up and all regression models were manually and automatically optimized."
- [methods] Automatic module selects linear or quadratic regression and appropriate standards: "Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards."
- [abstract] IS-corrected results were consistently lower by about a factor of two, suggesting matrix enhancement: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods. This suggests matrix enhancement upon LC-MS/MS analysis."
- [intro] Windows-only compatibility: "QuantyFey is compatible with **Windows** operating systems only."
