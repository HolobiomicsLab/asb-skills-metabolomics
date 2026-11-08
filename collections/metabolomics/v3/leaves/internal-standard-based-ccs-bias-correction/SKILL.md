---
name: internal-standard-based-ccs-bias-correction
description: Use when IM-MS lipidomics measurements (e.g., TIM-MS) are spiked with
  U13C-labeled yeast lipids and CCSN2 values show bias against the DTCCSN2 library
  that must be corrected per lipid class-adduct combination.
when_to_use_negative:
- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids detected —
  regression is impossible, so CCS correction cannot be applied to that combination
- The lipid class or adduct is outside the permitted combinations (e.g., AcCa, Co,
  LPE, PG, SPH entries are in the adduct table but not among the 10 correctable classes
  for correction)
- The goal is only to monitor/report CCS bias without correcting values — use MobiLipid_CCS-bias-calculation.Rmd
  instead of the correction workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: Automated R Markdown tool that computes CCS bias and lipid class-adduct-specific
    correction functions
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R 4.3.2) in which the R Markdown regression and correction
    are executed
  repo: https://cran.r-project.org/
- name: data.table
  role: R package used to load and filter the measured data and the DTCCSN2 library
    .csv files
  repo: https://cran.r-project.org/
- name: Skyline
  role: Upstream LC-IM-MS processing (LC peak integration, IM library creation, IM
    filtering) that produces the measured CCS values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:15048c599e3e8ada61e77f8ad72e207b2a45060c362ff68f515a91b606f01a4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R Markdown
  - Skyline
  - data.table
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# internal-standard-based-ccs-bias-correction

## Summary

An R Markdown–based procedure (MobiLipid) that generates lipid class- and adduct-specific linear CCS correction functions by regressing measured TIMCCSN2 values of U13C-labeled internal standard lipids against DTCCSN2 library values, then applies them to correct CCSN2 of all lipids in IM-MS lipidomics data.

## When to use

Use when IM-MS lipidomics measurements (e.g., TIM-MS) are spiked with U13C-labeled yeast lipids and CCSN2 values show bias against the DTCCSN2 library that must be corrected per lipid class-adduct combination.

## When NOT to use

- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids detected — regression is impossible, so CCS correction cannot be applied to that combination
- The lipid class or adduct is outside the permitted combinations (e.g., AcCa, Co, LPE, PG, SPH entries are in the adduct table but not among the 10 correctable classes for correction)
- The goal is only to monitor/report CCS bias without correcting values — use MobiLipid_CCS-bias-calculation.Rmd instead of the correction workflow

## Inputs

- Measured IM-MS lipidomics .csv with headers File, LipidClass, LipidSpecies, Adduct, Label (light/heavy), CCS
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values, Table S1)
- MobiLipid_CCS-bias-calculation_CCS-correction.Rmd R Markdown file

## Outputs

- Correction_functions.csv — table of slope m, intercept b, lipid class-adduct combination, and constituent lipid sets for all generated correction functions
- Corrected_CCS_values.csv — corrected CCS values and post-correction CCS bias per lipid and per correction function
- CCS_bias_no_correction, CCS_bias_mean_by_function, CCS_bias_mean_all_functions, Corrected_CCS_values_mean .csv tables
- .html/.pdf report (bias tables, violin plots, resampling summary) and .RData of all results

## How to apply

Load the measured data (.csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS) and U13C_DT_CCS_library.csv in R using data.table. Filter to 'heavy' (U13C-labeled) entries and restrict to the 10 correctable classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) using only permitted class-adduct combinations. For each combination, repeatedly draw distinct subsets of 3–6 labeled lipids (minimum 3 required) to build up to 100 distinct resamples, fitting y = m*x + b with measured CCS as x and DTCCSN2 library values as y. Write all fitted slopes/intercepts to Correction_functions.csv and apply the functions to correct CCSN2 of all lipids irrespective of labeling status; the rationale is internal standardization without additional external calibrants.

## Related tools

- **MobiLipid** (Automated R Markdown tool that computes CCS bias and lipid class-adduct-specific correction functions) — https://github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R 4.3.2) in which the R Markdown regression and correction are executed) — https://cran.r-project.org/
- **data.table** (R package used to load and filter the measured data and the DTCCSN2 library .csv files) — https://cran.r-project.org/
- **Skyline** (Upstream LC-IM-MS processing (LC peak integration, IM library creation, IM filtering) that produces the measured CCS values)

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation_CCS-correction.Rmd", output_format = c("html_document", "pdf_document"), output_file = paste0("MobiLipid_CCS-bias-calculation_CCS-correction_", filename, ".html"))
```

## Evaluation signals

- Post-correction mean absolute CCSN2 bias of U13C-labeled lipids drops toward the paper's reported ~0% per lipid class (from 0.78% positive / 0.33% negative mode pre-correction)
- Every fitted regression uses at least 3 distinct U13C-labeled lipids; resampling summary shows up to 100 distinct functions per class-adduct combination
- Correction_functions.csv contains slope/intercept pairs for all resamples, tagged with class-adduct combination and constituent lipid sets
- Nomenclature of LipidClass, LipidSpecies, and Adduct in the input .csv matches that of U13C_DT_CCS_library.csv, and only permitted class-adduct combinations are processed
- Output .html/.pdf and .RData are regenerated with per-function and per-lipid-count bias summaries (CCS_bias_mean_by_function, Corrected_CCS_values)

## Limitations

- CCS correction is restricted to 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) because a minimum of 3 labeled lipids per class-adduct combination is required
- Only 5 adducts are supported ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]) with class-specific restrictions
- No software version or changelog is provided for the MobiLipid repository

## Evidence

- [abstract] regression minimum of 3 labeled lipids per combination: [evidence span withheld — anchor status: not_found]
- [abstract] bias reduced to ~0%: [evidence span withheld — anchor status: not_found]
- [methods] correction functions per class-adduct combination: "MobiLipid allows for the correction of CCS values by computing correction function for each lipid class-adduct combinations"
- [readme] resampling and regression scheme: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression."
- [readme] regression input orientation: "Input values for linear regression using the equation *y* = *m* * *x* + *b* are measured CCS values of U<sup>13</sup>C labeled lipids as x value and <sup>DT</sup>CCS<sub>N2</sub> library values as y"
