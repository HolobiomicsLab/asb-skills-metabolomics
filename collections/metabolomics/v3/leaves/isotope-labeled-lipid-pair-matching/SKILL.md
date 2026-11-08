---
name: isotope-labeled-lipid-pair-matching
description: Use when DTIM-MS lipidomics data contain both natural ('light') and U13C-labeled
  ('heavy') versions of the same lipid species and you need to compute per-pair DTCCSN2
  bias against the U13C_DT_CCS_library.csv, e.
when_to_use_negative:
- Samples were not spiked with U13C-labeled internal standards (fully labeled yeast
  extract), so no 'heavy' entries exist to pair against the library
- The lipid class-adduct combination is outside the supported table (e.g. PC with
  [M+NH4] or AcCa with [M+NH4]) — such rows will not match library entries
- The goal is cross-technology bias monitoring (e.g. TIM-MS vs DTIM-MS) or CCS correction
  via regression functions rather than labeled/unlabeled pair matching
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MobiLipid
  role: R Markdown tool whose CCS-bias-calculation workflow implements the labeled/unlabeled
    pair matching and bias calculation against the DTCCSN2 library
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R Markdown
  role: Format of the automated MobiLipid scripts (MobiLipid_CCS-bias-calculation.Rmd)
    executed to compute and render the bias results
- name: R / RStudio
  role: Runtime environment (R 4.3.2, RStudio 2023.09.1) in which the R Markdown is
    rendered and the required packages (data.table, ggplot2, DT, knitr, ggbeeswarm,
    etc.) are installed
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:bbab7b8baeba306af3f5130883a03fb8e1ab4c1e5df19abbf688d62cbad8a4c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R / RStudio
  - R Markdown
  - RStudio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# isotope-labeled-lipid-pair-matching

## Summary

Matches U13C-labeled ('heavy') and unlabeled monoisotopic ('light') lipid pairs measured by DTIM-MS against the U13C yeast DTCCSN2 library to quantify the systematic CCS bias per lipid class-adduct combination. It establishes the internal-standardization baseline (≈0.25% labeled-vs-unlabeled shift) that underpins MobiLipid CCS quality control.

## When to use

Use when DTIM-MS lipidomics data contain both natural ('light') and U13C-labeled ('heavy') versions of the same lipid species and you need to compute per-pair DTCCSN2 bias against the U13C_DT_CCS_library.csv, e.g. to quantify the systematic labeled/unlabeled CCS offset or assess CCS quality control of an IM-MS run.

## When NOT to use

- Samples were not spiked with U13C-labeled internal standards (fully labeled yeast extract), so no 'heavy' entries exist to pair against the library
- The lipid class-adduct combination is outside the supported table (e.g. PC with [M+NH4] or AcCa with [M+NH4]) — such rows will not match library entries
- The goal is cross-technology bias monitoring (e.g. TIM-MS vs DTIM-MS) or CCS correction via regression functions rather than labeled/unlabeled pair matching

## Inputs

- Measured DTIM-MS data table as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values, Table S1)
- MobiLipid_CCS-bias-calculation.Rmd from github.com/FelinaHildebrand/MobiLipid

## Outputs

- CCS_bias_no_correction.csv — table of percent CCS bias before correction for each matched lipid
- Rendered .html and .pdf report with CCS bias tables and plots
- .RData (RDS) file storing all results as a list

## How to apply

Load the measured data table (.csv with headers 'File', 'LipidClass', 'LipidSpecies', 'Adduct', 'Label', 'CCS') and U13C_DT_CCS_library.csv in R/RStudio; use the Label column to separate 'light' from 'heavy' entries and match both to library entries on LipidClass–LipidSpecies–Adduct. Restrict matching to the valid lipid class-adduct combinations (e.g. PC with [M+H], [M+Na], [M+HCOO]; TG with [M+NH4], [M+Na]) using the library's nomenclature. For each matched heavy lipid compute the percent bias between measured CCS and the library DTCCSN2 value; repeat for matched light (monoisotopic) lipids against the same library entries. Aggregate per-lipid percent biases to obtain the mean systematic DTCCSN2 offset between U13C-labeled and unlabeled lipids. Run the MobiLipid R Markdown (MobiLipid_CCS-bias-calculation.Rmd) to render the bias table/plots and write CCS_bias_no_correction.csv; the labeled/unlabeled comparison should reproduce the ≈0.25% shift attributed to reduced-mass effects at moderate IM-MS resolution.

## Related tools

- **MobiLipid** (R Markdown tool whose CCS-bias-calculation workflow implements the labeled/unlabeled pair matching and bias calculation against the DTCCSN2 library) — https://github.com/FelinaHildebrand/MobiLipid
- **R Markdown** (Format of the automated MobiLipid scripts (MobiLipid_CCS-bias-calculation.Rmd) executed to compute and render the bias results)
- **R / RStudio** (Runtime environment (R 4.3.2, RStudio 2023.09.1) in which the R Markdown is rendered and the required packages (data.table, ggplot2, DT, knitr, ggbeeswarm, etc.) are installed) — https://cran.r-project.org/

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation.Rmd", output_format = c("html_document", "pdf_document"))  # run in RStudio after selecting data .csv and U13C_DT_CCS_library.csv via tcltk file choosers
```

## Evaluation signals

- Mean percent bias of the labeled/unlabeled comparison reproduces the small systematic DTCCSN2 shift of approximately 0.25% reported in Supporting Information Figure S5
- CCS_bias_no_correction.csv contains a percent bias entry for every matched lipid pair
- All matched pairs fall within the supported lipid class-adduct combinations (e.g. PC [M+H]/[M+Na]/[M+HCOO], TG [M+NH4]/[M+Na]) and use the library nomenclature
- Bias values are computed against the same library DTCCSN2 entry for both the light and heavy member of each pair
- Report renders both .html and .pdf with the CCS bias table and figure for each lipid class-adduct combination

## Limitations

- CCS correction functions via this workflow require a minimum of 3 U13C-labeled lipids per lipid class-adduct combination; CCS bias calculation alone is restricted to the supported adduct table (5 adducts, 15 lipid classes)
- The ≈0.25% labeled-vs-unlabeled shift is attributed to reduced-mass considerations observed with moderate resolution IM-MS, so its magnitude may vary with instrument resolving power
- No software version or changelog is specified for the MobiLipid repository

## Evidence

- [readme] Input CSV schema requirement: "The .csv file has to have the following headers: "File", "LipidClass", "LipidSpecies", "Adduct", "Label", "CCS""
- [readme] Label field semantics for light/heavy matching: ""Label": "light" for natural lipids and "heavy" for U<sup>13</sup>C labeled lipids"
- [methods] Bias calculation step definition: "calculate CCS bias between measured CCS values and library values of U<sup>13</sup>C labeled lipids"
- [intro] Cross-technology bias context in positive/negative mode: "the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies is 0.78% in positive ionization mode and 0.33% in negative ionization mode"
- [abstract] Minimum lipids required per class-adduct combination: "requiring only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes without requiring additional external measurements"
