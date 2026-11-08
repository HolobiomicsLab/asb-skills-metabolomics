---
name: ccs-reference-library-matching
description: 'Use when IM-MS lipidomics data spiked with U13C-labeled yeast internal
  standards needs CCS quality control: measured TIMCCSN2 or DTIMCCSN2 values must
  be matched to a DTCCSN2 reference library to quantify per-lipid CCS bias by ionization
  mode.'
when_to_use_negative:
- Samples are not spiked with U13C-labeled internal standard lipids from the fully
  labeled yeast extract, so no heavy entries exist to match against the DTCCSN2 library
- The goal is CCS correction (regression-based correction functions per lipid class-adduct),
  which requires the separate 'MobiLipid_CCS-bias-calculation_CCS-correction.Rmd'
  workflow
- Lipid class-adduct combinations fall outside the compatibility table (e.g. TG with
  [M+H]) or measured CCS values come from an IM technology/ion gas not covered by
  the N2-based DTCCSN2 library
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MobiLipid
  role: R Markdown workflow that performs the library matching and CCS bias calculation
    against U13C_DT_CCS_library.csv
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R 4.3.2) for executing the MobiLipid R Markdown bias
    calculation
  repo: https://cran.r-project.org/
- name: RStudio
  role: Environment (RStudio 2023.09.1) from which the MobiLipid R Markdown is launched
    and outputs are inspected
  repo: https://www.rstudio.com/products/rstudio/download/
- name: Skyline
  role: Upstream processing (version 23.1.0.268) that produces the measured CCS values
    imported for library matching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:5e9a86ce8c6e18db620cf3a1353cd8785a1442b3b3005cc49543996fee25002a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - RStudio
  - Skyline
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ccs-reference-library-matching

## Summary

Matches measured collision cross section (CCSN2) values of U13C-labeled lipids against the DTCCSN2 library (U13C_DT_CCS_library.csv) using library entries as reference, to compute per-lipid CCS bias (%) in ion mobility-MS lipidomics.

## When to use

Use when IM-MS lipidomics data spiked with U13C-labeled yeast internal standards needs CCS quality control: measured TIMCCSN2 or DTIMCCSN2 values must be matched to a DTCCSN2 reference library to quantify per-lipid CCS bias by ionization mode.

## When NOT to use

- Samples are not spiked with U13C-labeled internal standard lipids from the fully labeled yeast extract, so no heavy entries exist to match against the DTCCSN2 library
- The goal is CCS correction (regression-based correction functions per lipid class-adduct), which requires the separate 'MobiLipid_CCS-bias-calculation_CCS-correction.Rmd' workflow
- Lipid class-adduct combinations fall outside the compatibility table (e.g. TG with [M+H]) or measured CCS values come from an IM technology/ion gas not covered by the N2-based DTCCSN2 library

## Inputs

- Measured CCS data as .csv with headers 'File', 'LipidClass', 'LipidSpecies', 'Adduct', 'Label' ('light'/'heavy'), 'CCS' (e.g. from Skyline 23.1.0.268 processing of LC-IM-MS data)
- DTCCSN2 library for U13C-labeled yeast lipids ('U13C_DT_CCS_library.csv', 377 DTCCSN2 values, Table S1)

## Outputs

- CCS_bias_no_correction table (.csv) with per-lipid CCS bias (%) using library values as reference
- Mean absolute CCSN2 bias summarized by ionization mode (expected ~0.78% positive, ~0.33% negative for TIM-MS vs DTCCSN2)
- .html/.pdf MobiLipid report with CCS bias table and beeswarm figure, plus a .RData list of all results

## How to apply

In R/RStudio, run 'MobiLipid_CCS-bias-calculation.Rmd' via rmarkdown::render, selecting the .Rmd, the measured-data .csv (headers: File, LipidClass, LipidSpecies, Adduct, Label, CCS) and 'U13C_DT_CCS_library.csv' through tcltk::tk_choose.files dialogs. Restrict analysis to entries labeled 'heavy' (U13C-labeled internal standard lipids) and to lipid class-adduct combinations valid per the compatibility table (e.g. PC with [M+H], [M+Na], [M+HCOO]); adducts are limited to [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] with library-consistent nomenclature. Match each heavy lipid species-adduct to its DTCCSN2 library entry and compute per-lipid CCS bias (%) with the library value as reference, aggregating absolute biases into the CCS_bias_no_correction table and summarizing the mean absolute bias by ionization mode. The rationale is that DTCCSN2 library values from LC-DTIM-MS of a U13C yeast extract serve as internal calibrants chemically matched to analytes, avoiding dissimilar external calibrants.

## Related tools

- **MobiLipid** (R Markdown workflow that performs the library matching and CCS bias calculation against U13C_DT_CCS_library.csv) — https://github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R 4.3.2) for executing the MobiLipid R Markdown bias calculation) — https://cran.r-project.org/
- **RStudio** (Environment (RStudio 2023.09.1) from which the MobiLipid R Markdown is launched and outputs are inspected) — https://www.rstudio.com/products/rstudio/download/
- **Skyline** (Upstream processing (version 23.1.0.268) that produces the measured CCS values imported for library matching)

## Examples

```
Rmd <- tcltk::tk_choose.files(caption = "Select R markdown (.Rmd file)...", multi = FALSE); rmarkdown::render(Rmd, output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- Computed mean absolute CCSN2 bias per ionization mode reproduces the reported ~0.78% (positive) and ~0.33% (negative) for TIMCCSN2 vs DTCCSN2
- Imported .csv contains exactly the required headers 'File', 'LipidClass', 'LipidSpecies', 'Adduct', 'Label', 'CCS' with library-consistent lipid nomenclature
- Only 'Label' == 'heavy' entries and permitted lipid class-adduct combinations (e.g. Cer: [M+H], [M+Na], [M+HCOO]) enter the bias calculation
- Every heavy lipid species-adduct match resolves to a DTCCSN2 library entry (377-value U13C library) before bias computation
- Output artifacts CCS_bias_no_correction.csv, the .html/.pdf report, and the .RData result list are generated in the Rmd folder

## Limitations

- Only adducts [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] are supported, and not for every lipid class; the compatibility table must be respected
- Bias calculation alone does not correct CCS values; correction requires the CCS-correction Rmd and ≥3 U13C-labeled lipids per lipid class-adduct combination (restricted to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG)
- No software version is specified for MobiLipid and no repository changelog is provided
- The DTCCSN2 library is specific to U13C-labeled yeast lipids derived from the Neubauer et al. 2012 Pichia pastoris extract

## Evidence

- [results] Comparing TIMCCSN2 values of U13C-labeled lipids to DTCCSN2 library entries (DT as reference), the mean absolute CCSN2 bias is 0.78% in positive ionization mode and 0.33% in negative ionization mode.: "the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies is 0.78% in positive ionization mode and 0.33% in negative ionization mode"
- [readme] README specifies the input CSV schema and the heavy/light labeling convention used for library matching.: "The .csv file has to have the following headers: “File”, “LipidClass”, “LipidSpecies”, “Adduct”, “Label”, “CCS”"
- [readme] README describes the purpose of the bias-calculation Rmd against the library file.: "“MobiLipid_CCS-bias-calculation.Rmd” can be used to calculate the CCS bias between measured CCS values and"
- [abstract] Abstract establishes the reference library that matching is performed against.: "a newly established DT CCS N2 library for uniformly (U) 13 C-labeled lipids, derived from a U 13 C-labeled yeast extract, containing 377 DT CCS N2 values"
- [methods] Methods name the library CSV file imported for matching.: "the .csv file of the <sup>DT</sup>CCS<sub>N2</sub> library for U<sup>13</sup>C labeled lipids of yeast (“U13C_DT_CCS_library.csv”)"
