---
name: dtccs-library-lookup
description: Use when IM-MS lipidomics data are spiked with fully U13C-labeled yeast
  extract lipids and measured CCS values must be compared against a reference DTCCSN2
  library to quantify CCS bias per lipid class-adduct combination.
when_to_use_negative:
- The goal is to correct CCS values, not just look up biases — use MobiLipid_CCS-bias-calculation_CCS-correction.Rmd,
  which additionally computes linear correction functions per lipid class-adduct combination
  (requires >=3 U13C-labeled lipids).
- Input CCS values were not measured against the same lipid-class/adduct nomenclature
  or the adduct is not one of [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] supported for
  that lipid class.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: R Markdown tool that performs the library lookup, CCS bias calculation, and
    optional CCS correction against the DTCCSN2 library for U13C-labeled lipids
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R 4.3.2) in which the MobiLipid R Markdown is rendered
  repo: https://cran.r-project.org/
- name: RStudio
  role: IDE (RStudio 2023.09.1) used to run MobiLipid and inspect the resulting .RData
    results
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:2e3e48240728be655657db8652731c3e3b0e15c8fdb1e57cffb89891ee050ab6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R Markdown
  - RStudio
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# dtccs-library-lookup

## Summary

Look up reference DTCCSN2 values for U13C-labeled yeast lipids in the MobiLipid library (U13C_DT_CCS_library.csv, 377 values) to compute lipid class- and adduct-specific CCS biases between measured and library CCS values.

## When to use

Use when IM-MS lipidomics data are spiked with fully U13C-labeled yeast extract lipids and measured CCS values must be compared against a reference DTCCSN2 library to quantify CCS bias per lipid class-adduct combination.

## When NOT to use

- The goal is to correct CCS values, not just look up biases — use MobiLipid_CCS-bias-calculation_CCS-correction.Rmd, which additionally computes linear correction functions per lipid class-adduct combination (requires >=3 U13C-labeled lipids).
- Input CCS values were not measured against the same lipid-class/adduct nomenclature or the adduct is not one of [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] supported for that lipid class.

## Inputs

- Measured CCS data as .csv with headers: File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS (e.g. Example_data_import.csv)
- U13C_DT_CCS_library.csv: DTCCSN2 library for U13C-labeled yeast lipids (377 values, also Table S1)

## Outputs

- CCS_bias_no_correction.csv: table of CCS bias (%) before correction per lipid
- Rendered .html and .pdf report with CCS bias tables and plots per lipid class-adduct combination
- .RData file storing all results as a list (readable via readRDS())

## How to apply

Load the measured DTIM-MS data as a .csv with the exact headers File, LipidClass, LipidSpecies, Adduct, Label, CCS (Label = 'light' for natural lipids, 'heavy' for U13C-labeled) and the U13C_DT_CCS_library.csv containing the DTCCSN2 values for U13C-labeled yeast lipids. Restrict matching to the valid lipid class-adduct combinations (e.g. PC with [M+H], [M+Na], [M+HCOO]; TG with [M+NH4], [M+Na]; full table in the README), and match entries on LipidClass-LipidSpecies-Adduct using the library's nomenclature. For each matched heavy lipid, calculate the percent CCS bias between the measured CCS value and the library DTCCSN2 value, and likewise for matched light (monoisotopic) lipids against the same library entries, then aggregate per-lipid percent biases to obtain the mean systematic offset (e.g. the ~0.25% DTCCSN2 shift between U13C-labeled and unlabeled lipids). Run the MobiLipid R Markdown (MobiLipid_CCS-bias-calculation.Rmd) in R/RStudio, which renders bias tables and plots and writes CCS_bias_no_correction.csv.

## Related tools

- **MobiLipid** (R Markdown tool that performs the library lookup, CCS bias calculation, and optional CCS correction against the DTCCSN2 library for U13C-labeled lipids) — https://github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R 4.3.2) in which the MobiLipid R Markdown is rendered) — https://cran.r-project.org/
- **RStudio** (IDE (RStudio 2023.09.1) used to run MobiLipid and inspect the resulting .RData results) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation.Rmd", output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- The computed mean CCS bias for the labeled/unlabeled comparison reproduces the small systematic DTCCSN2 shift of ~0.25% reported in Supporting Information Figure S5.
- CCS_bias_no_correction.csv contains a percent bias entry for each matched lipid.
- All processed entries fall within the allowed lipid class-adduct combinations (e.g. PC: [M+H], [M+Na], [M+HCOO]; TG: [M+NH4], [M+Na]) and use the library's nomenclature.
- Bias magnitudes are consistent with cross-technology expectations: mean absolute CCSN2 bias of ~0.78% (positive mode) and ~0.33% (negative mode) when TIM CCSN2 values are compared to the DTCCSN2 library.

## Limitations

- The DTCCSN2 library covers only the 10 lipid classes supported for CCS correction (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) plus the other listed class-adduct combinations; lipids outside this scope cannot be looked up.
- CCS correction (beyond bias lookup) is restricted to lipid class-adduct combinations with a minimum of 3 U13C-labeled lipids for linear regression.
- No changelog is provided for the repository and no software version is specified for MobiLipid.
- The library was generated from DTIM-MS measurements of a U13C-labeled yeast extract; a systematic ~0.25% DTCCSN2 shift exists between U13C-labeled and unlabeled (monoisotopic) lipids, which must be accounted for when comparing light lipids directly to library values.

## Evidence

- [readme] Data import csv headers: "The .csv file has to have the following headers: "File", "LipidClass", "LipidSpecies", "Adduct", "Label", "CCS""
- [methods] Library file name: [evidence span withheld — anchor status: not_found]
- [methods] CCS bias calculation definition: [evidence span withheld — anchor status: not_found]
- [abstract] 377 library values: "a newly established DT CCS N2 library for uniformly (U) 13 C-labeled lipids, derived from a U 13 C-labeled yeast extract, containing 377 DT CCS N2 values"
- [readme] Minimum lipids for correction: "linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination"
- [readme] MobiLipid citation/DOI: "MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass Spectrometry Lipidomics by Internal Standardization"
