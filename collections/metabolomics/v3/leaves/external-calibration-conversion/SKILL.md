---
name: external-calibration-conversion
description: Use when you have already-integrated targeted LC-MS/MS peak areas (e.g.
  exported from MultiQuant) plus external calibration curves and need to compute concentrations
  per compound, optionally after drift correction.
when_to_use_negative:
- The data are not yet integrated — QuantyFey does not provide integration capabilities
  and expects already-integrated vendor data
- You need calibration via internal standard correction only when a suitable IS exists;
  unsuitable IS selection (e.g. 4-hydroxy-proline vs proline) introduces bias
- You are running on macOS — QuantyFey is compatible with Windows operating systems
  only (the Apptainer version is slow and difficult to set up there)
edam_operation: http://edamontology.org/operation_3441
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: QuantyFey
  role: Shiny application used to interactively or automatically optimize calibration
    regressions and convert integrated peak areas to concentrations with optional
    drift correction
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Implementation runtime for QuantyFey, including the Shiny framework (v1.10.0)
    and renv (v1.1.4) dependency management
- name: MultiQuant 3.0.3
  role: Vendor software used upstream for peak integration and manual review before
    export of peak areas and RTs to QuantyFey
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:96744bc242038f7ee2579d82a84025d061f873d90071768039d2129c6f99caf3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_3441
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# external-calibration-conversion

## Summary

Convert integrated targeted LC-MS/MS peak areas into analyte concentrations using compound-specific external calibration regression models, applying the areas as exported from vendor integration software without re-integration.

## When to use

Use when you have already-integrated targeted LC-MS/MS peak areas (e.g. exported from MultiQuant) plus external calibration curves and need to compute concentrations per compound, optionally after drift correction.

## When NOT to use

- The data are not yet integrated — QuantyFey does not provide integration capabilities and expects already-integrated vendor data
- You need calibration via internal standard correction only when a suitable IS exists; unsuitable IS selection (e.g. 4-hydroxy-proline vs proline) introduces bias
- You are running on macOS — QuantyFey is compatible with Windows operating systems only (the Apptainer version is slow and difficult to set up there)

## Inputs

- Already-integrated peak areas and retention times exported from vendor software (e.g. MultiQuant 3.0.3, Sciex) for a targeted LC-MS/MS sequence
- External calibration regression models per compound (with calibration standard levels and weights)

## Outputs

- Per-compound analyte concentrations in each sample derived from the calibration regression
- Method-comparison metrics (mean bias, limits of agreement, Passing-Bablok regression, Pearson and Spearman correlations of concentration vs injection position)

## How to apply

In QuantyFey, first optimize each compound's calibration regression model interactively (adjusting model form, weights, standard levels) or via the automatic module that selects a linear or quadratic fit and appropriate standards. Apply the fitted external calibration regression to the already-integrated peak areas, converting them to concentrations without any re-integration, as the app works as an additional software layer on vendor-exported data. If intensity drift is observed across the injection sequence, apply one of the supported corrections (IS correction, QC-based drift correction using a repeatedly injected non-blank sample with linear/loess/polynomial models, custom bracketing, or weighted bracketing) before or as part of the conversion. Note that IS-corrected concentrations in the porcine plasma evaluation were consistently lower by approximately a factor of two versus other methods, so review which correction strategy is compound-appropriate (e.g. 4-hydroxy-proline and methionine sulfoxide showed bias with unsuitable IS). Validate quantification accuracy by comparing methods via mean bias, limits of agreement, Passing-Bablok regression, and Pearson/Spearman correlations of concentration versus injection position.

## Related tools

- **QuantyFey** (Shiny application used to interactively or automatically optimize calibration regressions and convert integrated peak areas to concentrations with optional drift correction) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Implementation runtime for QuantyFey, including the Shiny framework (v1.10.0) and renv (v1.1.4) dependency management)
- **MultiQuant 3.0.3** (Vendor software used upstream for peak integration and manual review before export of peak areas and RTs to QuantyFey)

## Examples

```
Rscript -e "setwd('path_to_QuantyFey_app.r'); runApp('app.r')"  # or on Windows: QuantyFey.bat
```

## Evaluation signals

- Concentrations are produced from the already-integrated peak areas without re-integration, matching the vendor export (peak areas and RTs)
- Calibration regression form (linear or quadratic) and standard levels are justified by the interactive or automatic optimization output
- Chosen drift-correction strategy reduces drift: mean bias and limits of agreement improve, and Pearson/Spearman correlations of concentration vs injection position decrease versus uncorrected data
- For the evaluated porcine plasma dataset (25 compounds), IS-corrected results should not be silently accepted when they run ~2× lower than other methods (matrix enhancement signal)
- Windows-only deployment completed (QuantyFey.bat launched on Windows, or R 4.2.x/4.5.x local run) with renv-managed packages installed

## Limitations

- QuantyFey is compatible with Windows operating systems only; the Apptainer version is complicated and slow on macOS
- The app does not provide any integration capabilities — it works only on already-integrated mass spectrometry data
- Unsuitable IS selection for compounds like 4-hydroxy-proline and methionine sulfoxide introduced biases, so correction strategy must be chosen per compound
- Custom- or weighted bracketing methods improved quantification accuracy but showed variable performance across compounds
- Commercial platforms lack QC-based drift correction and quantification bracketing when IS are unavailable — hence this skill is needed beyond vendor tooling

## Evidence

- [methods] Works only on already-integrated data: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [intro] Windows-only compatibility: "QuantyFey is compatible with **Windows** operating systems only."
- [abstract] Upstream peak integration and export: "Peak integration and manual review of chromatographic data were performed in MultiQuant (v3.0.3, Sciex). Peak areas and retention times (RTs) were exported"
- [abstract] Drift correction strategies offered: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
- [abstract] IS-corrected concentrations ~2× lower: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods. This suggests matrix enhancement"
- [abstract] Unsuitable IS introduces bias: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases"
- [readme] Readme overview: visualization, analysis, quantification: "**QuantyFey** is a Shiny application for the **visualization, analysis, and quantification** of **mass spectrometry (MS) data** using **external calibration**."
- [readme] Automatic optimization of regression model and standards: "**Automatic Optimization Module** - automatic selection of **linear** or **quadratic** regression model, and selection of appropriate standards."
- [methods] Standalone version runs on R-portable 4.2.0 on Windows: "The standalone version QuantyFey (https://github.com/CDLMarkus/QuantyFey/releases/) only works on Windows systems. It runs on R-portable 4.2.0"
