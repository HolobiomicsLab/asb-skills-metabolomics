---
name: im-ms-lipidomics-ccs-quality-control
description: Use when TIM-MS (or another IM-MS platform) lipidomics data have been
  spiked with U13C-labeled yeast extract internal standards and you need to monitor
  and/or correct CCSN2 bias without measuring additional external calibrants.
when_to_use_negative:
- Input data have not been spiked with U13C-labeled yeast extract internal standards
  — the library-based bias calculation and correction cannot be applied
- A lipid class-adduct combination contains fewer than 3 U13C-labeled lipids — CCS
  correction via linear regression is not possible for that combination
- The adduct or lipid class of the input does not appear in the permitted table (e.g.
  AcCa with [M+NH4], SPH with [M-H]) or uses an adduct outside [M+H], [M+NH4], [M+Na],
  [M-H], [M+HCOO]
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: Fully automated R Markdown tool that integrates into IM-MS lipidomics workflows
    to calculate CCS bias and correct CCS values using the DTCCSN2 library for U13C-labeled
    lipids
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Programming environment in which MobiLipid is run; further data filtering,
    evaluation, and visualization
  repo: https://cran.r-project.org/
- name: RStudio
  role: Console/IDE used to select the .Rmd and data files, install required packages,
    and run the MobiLipid workflow
  repo: https://www.rstudio.com/products/rstudio/download/
- name: R Markdown
  role: Document format of the MobiLipid tool; rendered into .html and .pdf outputs
    containing CCS bias calculation and correction results
  repo: github.com/FelinaHildebrand/MobiLipid
- name: Skyline
  role: Upstream processing of LC-IM-MS data (LC peak integration, ion mobility library
    creation, ion mobility filtering) that produces the measured CCS values fed into
    MobiLipid
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:8213f7d457bc740a23bcd2a7f40db85d1c1dab03025f8979d7770384e9887696
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R Markdown
  - RStudio
  - Skyline
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# im-ms-lipidomics-ccs-quality-control

## Summary

Internal CCS quality control for IM-MS lipidomics that calculates CCS bias between measured TIM CCS N2 values and a DTCCSN2 library of U13C-labeled yeast lipids, then optionally corrects CCS values with lipid class- and adduct-specific linear correction functions.

## When to use

Use when TIM-MS (or another IM-MS platform) lipidomics data have been spiked with U13C-labeled yeast extract internal standards and you need to monitor and/or correct CCSN2 bias without measuring additional external calibrants.

## When NOT to use

- Input data have not been spiked with U13C-labeled yeast extract internal standards — the library-based bias calculation and correction cannot be applied
- A lipid class-adduct combination contains fewer than 3 U13C-labeled lipids — CCS correction via linear regression is not possible for that combination
- The adduct or lipid class of the input does not appear in the permitted table (e.g. AcCa with [M+NH4], SPH with [M-H]) or uses an adduct outside [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]

## Inputs

- .csv file of measured lipid CCS data with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS (e.g. Example_data_import.csv)
- .csv file of the DTCCSN2 library for U13C labeled yeast lipids (U13C_DT_CCS_library.csv, 377 DTCCSN2 values)
- IM-MS lipidomics measurement files spiked with fully U13C-labeled yeast extract internal standards
- One of the MobiLipid .Rmd files: MobiLipid_CCS-bias-calculation.Rmd or MobiLipid_CCS-bias-calculation_CCS-correction.Rmd

## Outputs

- CCS_bias_no_correction.csv — table with CCS bias (%) before correction
- Correction_functions.csv — table with all generated CCS correction functions
- CCS_bias_mean_by_function.csv — mean CCS bias per lipid class-adduct combination per correction function
- CCS_bias_mean_all_functions.csv — mean CCS bias per lipid class-adduct combination over all functions sharing the same number of lipids used to generate them
- Corrected_CCS_values.csv — corrected CCS values and CCS bias after correction for each lipid and each correction function
- Corrected_CCS_values_mean.csv — mean corrected CCS values per lipid over all functions with the same number of lipids used to generate the function
- Rendered .html and .pdf reports (plus a .RData/.RDS file storing all results as a list)

## How to apply

Import the measured lipid CCS data as a .csv with headers 'File', 'LipidClass', 'LipidSpecies', 'Adduct', 'Label', 'CCS' and the 'U13C_DT_CCS_library.csv' (377 DTCCSN2 values) into R via data.table. Calculate CCS bias between measured CCS values and library values of U13C-labeled lipids; pre-correction mean absolute biases are expected around 0.78% (positive) and 0.33% (negative) for TIMCCSN2 versus DTCCSN2. Optionally correct CCS values by computing linear correction functions (y = m*x + b) for each lipid class-adduct combination, using measured CCS values of U13C-labeled lipids as x and DTCCSN2 library values as y; each function requires a minimum of 3 labeled lipids, MobiLipid generates up to 100 distinct functions employing 3 to 6 lipids per combination, and correction is limited to the lipid classes Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG with the adducts [M+H], [M+NH4], [M+Na], [M-H], and [M+HCOO] (as permitted per class). After correction, all measured CCS values are corrected irrespective of their labeling status, and the residual bias against the DTCCSN2 library of U13C-labeled lipids is monitored. Render the results via rmarkdown::render into .html and .pdf reports and save intermediate tables as .csv files.

## Related tools

- **MobiLipid** (Fully automated R Markdown tool that integrates into IM-MS lipidomics workflows to calculate CCS bias and correct CCS values using the DTCCSN2 library for U13C-labeled lipids) — github.com/FelinaHildebrand/MobiLipid
- **R** (Programming environment in which MobiLipid is run; further data filtering, evaluation, and visualization) — https://cran.r-project.org/
- **RStudio** (Console/IDE used to select the .Rmd and data files, install required packages, and run the MobiLipid workflow) — https://www.rstudio.com/products/rstudio/download/
- **R Markdown** (Document format of the MobiLipid tool; rendered into .html and .pdf outputs containing CCS bias calculation and correction results) — github.com/FelinaHildebrand/MobiLipid
- **Skyline** (Upstream processing of LC-IM-MS data (LC peak integration, ion mobility library creation, ion mobility filtering) that produces the measured CCS values fed into MobiLipid)

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation_CCS-correction.Rmd", output_format = c("html_document", "pdf_document"), output_file = c(paste0(Rmd_name,"_",filename,".html"), paste0(Rmd_name,"_",filename,".pdf")))
```

## Evaluation signals

- Mean absolute CCSN2 bias of the 10 corrected lipid classes is reduced to approximately 0% (from pre-correction 0.78% positive / 0.33% negative)
- Every correction function used at least 3 and at most 6 U13C-labeled lipids; at most 100 distinct functions were generated per lipid class-adduct combination
- All 297 lipids across the 10 lipid classes show corrected CCSN2 values, irrespective of labeling status, matching Supporting Information tables (S2–S7)
- The .csv output tables (CCS_bias_no_correction, Correction_functions, CCS_bias_mean_by_function, CCS_bias_mean_all_functions, Corrected_CCS_values, Corrected_CCS_values_mean) exist and are populated
- Input .csv headers are exactly File, LipidClass, LipidSpecies, Adduct, Label, CCS, and every lipid class-adduct combination is one of the permitted pairs listed in the README table

## Limitations

- Requires a minimum of 3 U13C-labeled lipids within a lipid class-adduct combination to generate a correction function, restricting CCS correction to the 10 listed lipid classes
- The tool relies on the provided U13C_DT_CCS_library.csv; no additional external calibration measurements are supported, and the library contains only 377 DTCCSN2 values
- No changelog is provided for the repository and no software version is specified for MobiLipid
- Adduct-lipid class combinations are restricted (e.g. AcCa only [M+H]/[M-H], SPH only [M+H]); unsupported pairs cannot be corrected
- Claims of accurate lipid quantification are supported by external citations rather than demonstrated within this tool's scope

## Evidence

- [readme] Requires spiked U13C-labeled standards and library: [evidence span withheld — anchor status: not_found]
- [readme] Library provided with code: [evidence span withheld — anchor status: not_found]
- [readme] Bias calculation method: "first calculate CCS bias between measured CCS values and library values of U13C labeled lipids"
- [readme] Correction function construction: "computing correction function for each lipid class-adduct combinations"
- [readme] Minimum 3 lipids per function: "require a minimum of 3 lipids within a lipid class-adduct combination"
- [readme] Up to 100 functions from 3–6 lipids: "computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination"
- [readme] Linear regression setup: [evidence span withheld — anchor status: not_found]
- [readme] Correct all values regardless of label: "all measured CCS values are corrected, irrespective of their labeling status"
- [abstract] Pre-correction biases in abstract: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [abstract] 10 lipid classes reduced to ~0% bias: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [abstract] Minimum 3 labeled lipids across 10 classes: "requiring only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes"
- [intro] R version and IDE: "R version 4.3.2 and RStudio 2023.09.1"
- [intro] No additional external calibration needed: "utilized for internal CCS monitoring and correction without the requirement for additional external calibration data to be measured"
- [intro] Library filter criteria: "a CCS relative standard deviation <1%"
- [readme] Rendered output formats: "generates a .html and a .pdf file with all data of CCS bias calculation and correction"
