---
name: isotope-labeled-internal-standardization
description: Use when LC-IM-MS lipidomics samples are spiked with U13C-labeled internal
  standards and you must quantify or correct CCSN2 bias against a DT CCS library without
  extra external calibrant measurements.
when_to_use_negative:
- Samples are not spiked with U13C-labeled internal standard lipids (e.g. only deuterated
  lipid mixes like EquiSPLASH LIPIDOMIX, which offer a limited lipid count)
- The lipid class-adduct combination in the data is not supported by MobiLipid's compatibility
  table, or fewer than 3 heavy lipids exist per combination so linear correction cannot
  be fit
- The measured data .csv lacks the required headers or its lipid class/species nomenclature
  differs from the DTCCSN2 library
tools:
- name: MobiLipid
  role: R Markdown tool that performs CCS bias calculation and lipid class-adduct-specific
    CCS correction against the U13C DTCCSN2 library
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R 4.3.2) hosting the MobiLipid R Markdown and required
    packages (rmarkdown, data.table, ggplot2, tcltk, etc.)
  repo: https://cran.r-project.org/
- name: RStudio
  role: Programming environment (RStudio 2023.09.1) used for data filtering, evaluation,
    visualization, and running the Rmd via console code
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:5a794a244988179998ec1170e70367aca5ac601a6ee59590d9c8a4ed4c6218db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - RStudio
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# isotope-labeled-internal-standardization

## Summary

Assess and correct CCS bias in IM-MS lipidomics by comparing measured CCS values of U13C-labeled yeast lipids against a DTCCSN2 reference library, using the MobiLipid R Markdown tool for lipid class- and adduct-specific internal standardization.

## When to use

Use when LC-IM-MS lipidomics samples are spiked with U13C-labeled internal standards and you must quantify or correct CCSN2 bias against a DT CCS library without extra external calibrant measurements.

## When NOT to use

- Samples are not spiked with U13C-labeled internal standard lipids (e.g. only deuterated lipid mixes like EquiSPLASH LIPIDOMIX, which offer a limited lipid count)
- The lipid class-adduct combination in the data is not supported by MobiLipid's compatibility table, or fewer than 3 heavy lipids exist per combination so linear correction cannot be fit
- The measured data .csv lacks the required headers or its lipid class/species nomenclature differs from the DTCCSN2 library

## Inputs

- Measured IM-MS lipidomics data as .csv with headers 'File', 'LipidClass', 'LipidSpecies', 'Adduct', 'Label', 'CCS' (Label = 'light'/'heavy'; adducts limited to [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO])
- DTCCSN2 library for U13C-labeled yeast lipids ('U13C_DT_CCS_library.csv', 377 DT CCS N2 values; Table S1)
- MobiLipid R Markdown files 'MobiLipid_CCS-bias-calculation.Rmd' and/or 'MobiLipid_CCS-bias-calculation_CCS-correction.Rmd'

## Outputs

- CCS_bias_no_correction.csv: table of per-lipid CCS bias (%) before correction
- HTML and PDF reports (plus .RData list) with bias tables, beeswarm/violin plots, correction functions, and mean bias before vs after correction per lipid class-adduct combination
- With correction Rmd: Correction_functions.csv, CCS_bias_mean_by_function.csv, CCS_bias_mean_all_functions.csv, Corrected_CCS_values.csv, Corrected_CCS_values_mean.csv

## How to apply

Run 'MobiLipid_CCS-bias-calculation.Rmd' in RStudio (R 4.3.2) via rmarkdown::render, selecting the .Rmd, the measured-data .csv (headers: File, LipidClass, LipidSpecies, Adduct, Label, CCS), and 'U13C_DT_CCS_library.csv' through three tcltk::tk_choose.files dialogs. Restrict analysis to rows labeled 'heavy' (U13C-labeled internal standard lipids) and to lipid class-adduct combinations valid per the compatibility table (e.g. PC with [M+H], [M+Na], [M+HCOO]; TG only with [M+NH4], [M+Na]). Match each heavy species-adduct to its DTCCSN2 library entry, compute per-lipid CCS bias (%) with the library value as reference, and aggregate into the CCS_bias_no_correction table summarized by ionization mode. Optionally use 'MobiLipid_CCS-bias-calculation_CCS-correction.Rmd' to build linear regression correction functions (y = m*x + b; measured CCS as x, library CCS as y) from 3–6 heavy lipids per class-adduct combination, computing up to 100 distinct functions per combination and correcting all CCS values irrespective of labeling status.

## Related tools

- **MobiLipid** (R Markdown tool that performs CCS bias calculation and lipid class-adduct-specific CCS correction against the U13C DTCCSN2 library) — github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R 4.3.2) hosting the MobiLipid R Markdown and required packages (rmarkdown, data.table, ggplot2, tcltk, etc.)) — https://cran.r-project.org/
- **RStudio** (Programming environment (RStudio 2023.09.1) used for data filtering, evaluation, visualization, and running the Rmd via console code) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
Rmd <- tcltk::tk_choose.files(caption = "Select R markdown...", multi = FALSE); rmarkdown::render(paste0(Rmd), output_format = c("html_document", "pdf_document"), output_file = paste0(Rmd_name, "_", filename, ".html"))
```

## Evaluation signals

- Mean absolute CCSN2 bias per ionization mode should reproduce ~0.78% in positive and ~0.33% in negative ionization mode when TIM CCS values are referenced to the DTCCSN2 library
- Input .csv must contain exactly the headers File, LipidClass, LipidSpecies, Adduct, Label, CCS, and nomenclature must match the DTCCSN2 library
- Only entries with Label == 'heavy' contribute to bias calculation; light lipids are excluded before matching to library entries
- Correction functions require a minimum of 3 lipids per lipid class-adduct combination; combinations below this threshold are reported as not correctable
- After correction, mean absolute CCSN2 biases of 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) should be reduced to approximately 0%

## Limitations

- No changelog and no software version specified for MobiLipid in the repository
- CCS correction is restricted to 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) requiring at least 3 U13C-labeled lipids per class-adduct combination
- Adducts are limited to [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] with class-specific compatibility; unsupported combinations are excluded
- Claims of accurate lipid quantification are supported by external citations rather than demonstrated within the paper's evaluation

## Evidence

- [abstract] mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [intro] the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies: "the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies is 0.78% in positive ionization mode and 0.33% in negative ionization mode"
- [readme] requires a .csv file with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS: "The .csv file has to have the following headers: “File”, “LipidClass”, “LipidSpecies”, “Adduct”, “Label”, “CCS”"
- [readme] correction functions require a minimum of 3 lipids within a lipid class-adduct combination: "linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination which restricts the CCS correction"
- [readme] correction restricted to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG: "restricts the CCS correction to the following lipid classes: Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG"
- [methods] calculate CCS bias between measured CCS values and library values of U13C labeled lipids: "calculate CCS bias between measured CCS values and library values of U<sup>13</sup>C labeled lipids"
- [abstract] CCSN2 biases of 10 lipid classes could be reduced to approximately 0%: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
