---
name: ccs-correction-function-regression
description: Use when IM-MS lipidomics data spiked with U13C-labeled yeast internal
  standards show CCSN2 bias versus the DTCCSN2 library, and lipid class-adduct combinations
  contain ≥3 U13C-labeled lipids for regression-based correction.
when_to_use_negative:
- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids detected —
  linear regression cannot be fitted, and correction is restricted to the 10 correctable
  classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) and their permitted adducts
  ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO])
- Samples lack U13C-labeled yeast internal standards entirely — use MobiLipid_CCS-bias-calculation.Rmd
  for bias calculation only, since the correction functions rely on labeled lipids
  as regression inputs
- Input CCS values come from external calibration measurements rather than the workflow's
  internal standardization approach — the tool explicitly eliminates the need for
  additional external calibration besides vendor-specific requirements
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: R Markdown tool (MobiLipid_CCS-bias-calculation_CCS-correction.Rmd) that performs
    the resampling, linear regression, and CCS correction
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Programming environment (R version 4.3.2) in which data filtering, regression,
    evaluation, and visualization are executed; packages include data.table, rmarkdown,
    ggplot2
  repo: https://cran.r-project.org/
- name: R Markdown
  role: Automated execution format rendering results to HTML/PDF output files
  repo: https://github.com/FelinaHildebrand/MobiLipid
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:59475a03c07d06c87e5cf672fda0a474973890558bf271bdf091ad2c258f1d5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R Markdown
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ccs-correction-function-regression

## Summary

Generates lipid class- and adduct-specific linear CCS correction functions by regressing measured TIMCCSN2 values of U13C-labeled lipids (3–6 per subset, up to 100 resamples) against the DTCCSN2 library values, then applies the functions to correct CCSN2 values of all lipids in that class-adduct combination.

## When to use

Use when IM-MS lipidomics data spiked with U13C-labeled yeast internal standards show CCSN2 bias versus the DTCCSN2 library, and lipid class-adduct combinations contain ≥3 U13C-labeled lipids for regression-based correction.

## When NOT to use

- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids detected — linear regression cannot be fitted, and correction is restricted to the 10 correctable classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) and their permitted adducts ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO])
- Samples lack U13C-labeled yeast internal standards entirely — use MobiLipid_CCS-bias-calculation.Rmd for bias calculation only, since the correction functions rely on labeled lipids as regression inputs
- Input CCS values come from external calibration measurements rather than the workflow's internal standardization approach — the tool explicitly eliminates the need for additional external calibration besides vendor-specific requirements

## Inputs

- Measured IM-MS lipidomics data as .csv with required headers: File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS (e.g. Example_data_import.csv)
- U13C_DT_CCS_library.csv — the DTCCSN2 library for U13C-labeled yeast lipids (377 values, also provided as Table S1)

## Outputs

- Correction_functions.csv — table of all generated CCS correction functions (slope m, intercept b, lipid class-adduct combination, constituent lipid sets)
- Corrected_CCS_values.csv — corrected CCS values and post-correction CCS bias for each lipid and each correction function
- CCS_bias_mean_by_function.csv and CCS_bias_mean_all_functions.csv — mean CCS bias per combination per function and aggregated over all functions
- HTML/PDF report (.html and .pdf via rmarkdown::render) plus an .RData list of all results readable with readRDS()

## How to apply

Load the measured data (.csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS) and the U13C_DT_CCS_library.csv reference using data.table in R, then filter to 'heavy' (U13C-labeled) entries restricted to the 10 correctable lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) and their permitted adducts. For each lipid class-adduct combination, repeatedly draw distinct subsets of 3 to 6 labeled lipids (a minimum of 3 is required for regression), computing up to 100 distinct resamples per combination. For each subset, fit the linear model y = m*x + b with measured CCS values of the U13C-labeled lipids as x and DTCCSN2 library values as y, then collect all fitted slope/intercept parameters with their combination and constituent lipid sets into Correction_functions.csv. After generating all functions, correct all measured CCS values irrespective of labeling status, and validate by confirming the CCS bias between corrected values and library values of the labeled lipids is reduced toward the reported outcome of approximately 0% mean absolute bias per lipid class (from a pre-correction baseline of 0.78% positive / 0.33% negative mode).

## Related tools

- **MobiLipid** (R Markdown tool (MobiLipid_CCS-bias-calculation_CCS-correction.Rmd) that performs the resampling, linear regression, and CCS correction) — https://github.com/FelinaHildebrand/MobiLipid
- **R** (Programming environment (R version 4.3.2) in which data filtering, regression, evaluation, and visualization are executed; packages include data.table, rmarkdown, ggplot2) — https://cran.r-project.org/
- **R Markdown** (Automated execution format rendering results to HTML/PDF output files) — https://github.com/FelinaHildebrand/MobiLipid

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation_CCS-correction.Rmd", output_format = c("html_document", "pdf_document"), output_file = paste0("MobiLipid_CCS-bias-calculation_CCS-correction_", filename, ".html"))
```

## Evaluation signals

- Correction_functions.csv contains fitted slope m and intercept b for each lipid class-adduct combination with the constituent U13C-labeled lipid sets, and no function was built from fewer than 3 lipids
- Post-correction mean absolute CCSN2 bias of the U13C-labeled lipids is reduced toward approximately 0% per lipid class (baseline 0.78% positive / 0.33% negative mode)
- Resampling summary shows up to 100 distinct correction functions per combination employing 3 to 6 lipids each
- Number-of-lipids output table reports which lipid class-adduct combinations had sufficient labeled lipids for correction
- All measured CCS values (light and heavy) appear in Corrected_CCS_values.csv with per-function corrected values and bias

## Limitations

- CCS correction is restricted to the 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) and specific adducts per class; other combinations (e.g. AcCa, Co, LPE, PG, SPH) support bias calculation only
- A minimum of 3 U13C-labeled lipids per lipid class-adduct combination is mandatory; combinations with fewer labeled lipids cannot be corrected
- No changelog and no software version specified for MobiLipid (only R 4.3.2 / RStudio 2023.09.1 are pinned)

## Evidence

- [readme] MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression.: "MobiLipid computes up to 100 distinct correction
functions employing 3 to 6 lipids of a lipid class-adduct combination for
linear regression."
- [readme] Correction function regression inputs and equation, with correction applied to all measured CCS values: "Input values for linear regression using
the equation *y* = *m* * *x* + *b* are measured CCS values of U<sup>13</sup>C labeled lipids as x value and <sup>DT</sup>CCS<sub>N2</sub> library values as y"
- [abstract] Minimum 3 labeled lipids per combination across 10 lipid classes: [evidence span withheld — anchor status: not_found]
- [abstract] Correction reduces mean absolute CCSN2 biases of 10 lipid classes to approximately 0%: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [intro] Pre-correction baseline bias between TIM and DTIM technologies for labeled lipids: "the mean absolute CCSN2 bias of U<sup>13</sup>C-labeled lipids between the two IM technologies
is 0.78% in positive ionization mode and 0.33% in negative ionization
mode"
