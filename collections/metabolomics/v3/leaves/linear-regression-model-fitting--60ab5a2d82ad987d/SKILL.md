---
name: linear-regression-model-fitting
description: Use when IM-MS lipidomics data (e.g., TIMCCSN2 values from a Bruker timsTOF
  Pro) are spiked with U13C-labeled lipids and a DTCCSN2 reference library exists,
  and CCS biases must be corrected per lipid class-adduct combination via linear regression.
when_to_use_negative:
- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids detected —
  regression cannot be fit and CCS correction is restricted to Cer, DG, HexCer, LPC,
  PA, PC, PE, PI, PS, and TG only
- The data are not spiked with U13C-labeled internal standards (fully labeled yeast
  extract) or no DTCCSN2 library values are available — the x and y regression inputs
  do not exist
- The goal is only CCS bias monitoring without correction — use MobiLipid_CCS-bias-calculation.Rmd
  instead of the correction workflow
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MobiLipid
  role: R Markdown tool that automates the resampling, linear regression fitting,
    and class-adduct-specific CCS correction
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R 4.3.2
  role: Runtime environment in which the regression and correction R Markdown is rendered
  repo: https://cran.r-project.org/
- name: data.table
  role: R package used to load and filter the measured-data .csv and the U13C_DT_CCS_library.csv
    before regression
- name: RStudio 2023.09.1
  role: Environment for installing packages via the console and running the MobiLipid
    R Markdown
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:50eba8c39ef7b07b32c6808bf80e4ef8fede13c99f156ba3ff63d9c515c0c80a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R 4.3.2
  - R Markdown
  - RStudio 2023.09.1
  - data.table
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# linear-regression-model-fitting

## Summary

Fits lipid class- and adduct-specific linear CCS correction functions (y = m*x + b) by regressing measured TIMCCSN2 values of U13C-labeled yeast lipids against DTCCSN2 library values, enabling internal standardization-based CCS bias correction in IM-MS lipidomics.

## When to use

Use when IM-MS lipidomics data (e.g., TIMCCSN2 values from a Bruker timsTOF Pro) are spiked with U13C-labeled lipids and a DTCCSN2 reference library exists, and CCS biases must be corrected per lipid class-adduct combination via linear regression.

## When NOT to use

- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids detected — regression cannot be fit and CCS correction is restricted to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG only
- The data are not spiked with U13C-labeled internal standards (fully labeled yeast extract) or no DTCCSN2 library values are available — the x and y regression inputs do not exist
- The goal is only CCS bias monitoring without correction — use MobiLipid_CCS-bias-calculation.Rmd instead of the correction workflow

## Inputs

- Measured IM-MS lipidomics data as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS (measured CCS value)
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 DTCCSN2 values), shipped with the MobiLipid repository
- MobiLipid_CCS-bias-calculation_CCS-correction.Rmd R Markdown script

## Outputs

- Correction_functions.csv — table of all fitted linear correction functions (slope m, intercept b) with lipid class-adduct combination and constituent lipid sets
- CCS_bias_no_correction — table of CCS bias (%) before correction
- CCS_bias_mean_by_function and CCS_bias_mean_all_functions — mean CCS bias per combination per function / across functions
- Corrected_CCS_values and Corrected_CCS_values_mean — corrected CCS values and post-correction bias per lipid and per function
- .html and .pdf reports plus an .RData list object readable via readRDS()

## How to apply

Filter the imported .csv data (headers: File, LipidClass, LipidSpecies, Adduct, Label, CCS) to 'heavy' (U13C-labeled) entries and restrict to the 10 correctable lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) using only permitted class-adduct combinations (e.g., PC [M+H], [M+Na], [M+HCOO]). For each combination, repeatedly draw distinct subsets of 3 to 6 labeled lipids (minimum 3 required for regression), up to 100 distinct resamples per combination. Fit y = m*x + b with measured CCS values of the U13C-labeled lipids as x and DTCCSN2 library values as y, collecting slope m and intercept b per function. Apply every fitted function to correct all measured CCS values irrespective of labeling status, then compute the bias between corrected CCS values and the DTCCSN2 library for labeled lipids to assess the correction. Rationale: a U13C-labeled yeast extract and the provided U13C_DT_CCS_library.csv (377 DTCCSN2 values) eliminate the need for additional external CCS calibrants, and 3–6 lipids per regression keep functions class- and adduct-specific.

## Related tools

- **MobiLipid** (R Markdown tool that automates the resampling, linear regression fitting, and class-adduct-specific CCS correction) — https://github.com/FelinaHildebrand/MobiLipid
- **R 4.3.2** (Runtime environment in which the regression and correction R Markdown is rendered) — https://cran.r-project.org/
- **data.table** (R package used to load and filter the measured-data .csv and the U13C_DT_CCS_library.csv before regression)
- **RStudio 2023.09.1** (Environment for installing packages via the console and running the MobiLipid R Markdown)

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation_CCS-correction.Rmd", output_format = "html_document")
```

## Evaluation signals

- The bias between corrected CCS values and DTCCSN2 library values of U13C-labeled lipids should drop toward approximately 0% mean absolute bias per lipid class (paper reports 10 lipid classes reduced to ~0% from 0.78% positive / 0.33% negative mode pre-correction)
- Correction_functions.csv must contain up to 100 distinct resampled functions per lipid class-adduct combination, each with a valid slope m and intercept b
- Every fitted function must be built from at least 3 and at most 6 distinct U13C-labeled lipids; combinations with fewer than 3 labeled lipids must be flagged as not correctable
- Corrected_CCS_values must include entries for all measured lipids (both 'light' and 'heavy'), since correction is applied irrespective of labeling status
- Output must render as .html and .pdf with violin plots of CCS bias before and after correction grouped by number of labeled lipids used per function

## Limitations

- CCS correction is restricted to the 10 correctable lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG); classes such as AcCa, Co, LPE, PG, SPH support bias calculation but not correction
- Only 5 adducts are supported ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]) and not all adducts are valid for every lipid class
- In the .pdf output only the first 10 rows of tables are displayed; full tables must be accessed via the .html file
- No software version or changelog is specified for MobiLipid, so exact regression implementation details must be inspected in the .Rmd source

## Evidence

- [readme] CCS correction functions are based on linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination: "CCS correction functions are
based on linear regression functions which require a minimum of 3 lipids
within a lipid class-adduct combination"
- [readme] MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression: "MobiLipid computes up to 100 distinct correction
functions employing 3 to 6 lipids of a lipid class-adduct combination
for linear regression"
- [methods] MobiLipid allows for the correction of CCS values by computing correction function for each lipid class-adduct combinations: "MobiLipid allows for the correction of CCS values by computing correction function for each lipid class-adduct combinations"
- [abstract] the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [intro] requiring only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes without requiring additional external measurements: "requiring only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes without requiring additional external measurements"
