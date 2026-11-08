---
name: resampling-based-correction-function-generation
description: Use when IM-MS lipidomics CCS values need class- and adduct-specific
  correction and ≥3 U13C-labeled lipids of a lipid class-adduct combination (e.g.,
  PC [M+H]+) are detected in measured data.
when_to_use_negative:
- Fewer than 3 U13C-labeled lipids are detected for a lipid class-adduct combination
  — linear regression correction is not possible and CCS correction is skipped
- Only CCS bias monitoring (no correction) is needed — use MobiLipid_CCS-bias-calculation.Rmd
  instead of the CCS-correction workflow
- Lipid class-adduct combination is outside the supported set (e.g., LPE with [M+HCOO])
  — MobiLipid restricts correction to specific class-adduct pairs
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MobiLipid
  role: R Markdown tool that performs resampling of 3–6 U13C-labeled lipids to generate
    lipid class- and adduct-specific CCS correction functions
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Execution environment for the MobiLipid R Markdown pipeline
  repo: https://cran.r-project.org/
- name: RStudio
  role: IDE in which the MobiLipid .Rmd is rendered via rmarkdown::render
  repo: https://www.rstudio.com/products/rstudio/download/
- name: rmarkdown
  role: Renders the CCS-correction .Rmd to HTML/PDF outputs
- name: tcltk
  role: File chooser dialogs for selecting the .Rmd, measured-data CSV, and U13C_DT_CCS_library.csv
- name: ggplot2
  role: Violin/plot visualization of CCS bias before and after correction
- name: data.table
  role: Data handling for measured CCS data and library tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:12a74b72466eaac0745c083171adab1882ef58a4ce1ececdd759b89876f086e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R Markdown
  - RStudio
  - data.table
  - ggbeeswarm
  - ggplot2
  - rmarkdown
  - tcltk
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# resampling-based-correction-function-generation

## Summary

Generates up to 100 lipid class- and adduct-specific CCS correction functions in MobiLipid by resampling 3–6 U13C-labeled lipids per combination into linear regressions (y = m*x + b) mapping measured CCS to DTCCSN2 library values, enabling robust internal CCS correction without external calibrants.

## When to use

Use when IM-MS lipidomics CCS values need class- and adduct-specific correction and ≥3 U13C-labeled lipids of a lipid class-adduct combination (e.g., PC [M+H]+) are detected in measured data.

## When NOT to use

- Fewer than 3 U13C-labeled lipids are detected for a lipid class-adduct combination — linear regression correction is not possible and CCS correction is skipped
- Only CCS bias monitoring (no correction) is needed — use MobiLipid_CCS-bias-calculation.Rmd instead of the CCS-correction workflow
- Lipid class-adduct combination is outside the supported set (e.g., LPE with [M+HCOO]) — MobiLipid restricts correction to specific class-adduct pairs

## Inputs

- .csv of measured lipid CCS values with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS (Label = 'light' or 'heavy')
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values)
- MobiLipid_CCS-bias-calculation_CCS-correction.Rmd

## Outputs

- Correction_functions.csv — table of all generated CCS correction functions with regression slopes and intercepts
- CCS_bias_mean_by_function.csv — mean CCS bias per lipid class-adduct combination for each correction function
- CCS_bias_mean_all_functions.csv — mean CCS bias pooled over functions with the same U13C-labeled lipid count
- Corrected_CCS_values.csv and Corrected_CCS_values_mean.csv — corrected CCS values and post-correction bias per lipid and per function
- HTML/PDF report with violin plots of CCS bias before/after correction and an .RData list of all results

## How to apply

Run MobiLipid_CCS-bias-calculation_CCS-correction.Rmd (rendered via rmarkdown::render in RStudio) after importing the measured-data CSV with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS and the U13C_DT_CCS_library.csv. For each lipid class-adduct combination, resample 3 to 6 U13C-labeled ('heavy') lipids per function — respecting the linear-regression minimum of 3 lipids — to build up to 100 distinct correction functions in which measured CCS values are the x inputs and DTCCSN2 library values are the y targets (y = m*x + b). Rank the resulting functions by regression slope deviation from 1 (outliers with slopes like 0.01, 0.46, or 2.42 signal unreliable functions) and by post-correction CCS bias distributions. Apply every function to all measured CCS values regardless of labeling status, then summarize mean bias per function (CCS_bias_mean_by_function) and pooled by lipid count (CCS_bias_mean_all_functions); more U13C-labeled lipids per function yields more robust correction, with biases approaching ~0%.

## Related tools

- **MobiLipid** (R Markdown tool that performs resampling of 3–6 U13C-labeled lipids to generate lipid class- and adduct-specific CCS correction functions) — github.com/FelinaHildebrand/MobiLipid
- **R** (Execution environment for the MobiLipid R Markdown pipeline) — https://cran.r-project.org/
- **RStudio** (IDE in which the MobiLipid .Rmd is rendered via rmarkdown::render) — https://www.rstudio.com/products/rstudio/download/
- **rmarkdown** (Renders the CCS-correction .Rmd to HTML/PDF outputs)
- **tcltk** (File chooser dialogs for selecting the .Rmd, measured-data CSV, and U13C_DT_CCS_library.csv)
- **ggplot2** (Violin/plot visualization of CCS bias before and after correction)
- **data.table** (Data handling for measured CCS data and library tables)

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation_CCS-correction.Rmd", output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- Regression slopes of known outlier functions match reported values (e.g., functions 31, 93, 98 for PC [M+H]+ with slopes 0.01, 0.46, 2.42)
- Mean absolute post-correction CCS bias for functions built from more U13C-labeled lipids (4–6) is lower than for 3-lipid functions, approaching ~0%
- Every generated function uses between 3 and 6 lipids per regression; count of functions per combination ≤ 100
- Corrected_CCS_values.csv contains bias entries for each lipid under every function, and corrected bias for U13C-labeled lipids is reduced relative to CCS_bias_no_correction
- Output artifacts (.html, .pdf, Correction_functions, CCS_bias_mean_by_function, CCS_bias_mean_all_functions, Corrected_CCS_values_mean) are all generated with the expected file names

## Limitations

- Correction is restricted to lipid classes with at least 3 detected U13C-labeled lipids per class-adduct combination (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG among others); combinations with fewer lipids are skipped
- Resampling relies on 3–6 lipids; functions built from only 3 lipids can be unstable (slope deviations as large as 2.42 from 1)
- Supported adducts are limited to [M+H], [M+NH4], [M+Na], [M-H], and [M+HCOO] with class-specific restrictions
- Input nomenclature for LipidClass and LipidSpecies must match the DTCCSN2 library exactly or entries will not join
- No explicit software version for MobiLipid is specified; claims rely on the Anal. Chem. 2024 publication

## Evidence

- [readme] Resampling 3–6 lipids per function to build up to 100 correction functions: "MobiLipid computes up to 100 distinct correction
    functions employing 3 to 6 lipids of a lipid class-adduct combination
    for linear regression."
- [readme] Linear regression with measured CCS as x and library as y: "Input values for linear regression using the
    equation *y* = *m* \* *x* + *b* are measured CCS values of
    U<sup>13</sup>C labeled lipids as x value and
    <sup>DT</sup>CCS<sub>N2</sub> library"
- [readme] Minimum of 3 lipids restricts correction to specific lipid classes: "CCS correction functions are
based on linear regression functions which require a minimum of 3 lipids
within a lipid class-adduct combination which restricts the CCS
correction to the following lipid"
- [readme] Correction applied to all measured CCS values: "After
generation of all CCS correction functions, all measured CCS values are
corrected, irrespective of their labeling status."
- [results] Functions 31, 93, 98 show subpar bias distributions and slopes deviating from 1: [evidence span withheld — anchor status: not_found]
- [results] More U13C-labeled lipids per function lead to more robust correction: [evidence span withheld — anchor status: not_found]
- [readme] Output artifacts from the correction run: "CCS_bias_mean_by_function: Table with the mean CCS bias for each
    lipid class-adduct combination for each correction function"
- [results] Purpose: verifying robustness of correction functions: [evidence span withheld — anchor status: not_found]
