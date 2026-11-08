---
name: ccs-bias-calculation-ion-mobility-lipidomics
description: Use when LC-IM-MS lipidomics samples (e.g., from a Bruker timsTOF Pro
  TIM-MS or Agilent 6560 DTIM-MS) are spiked with U13C-labeled yeast extract standards
  and you need to quantify CCSN2 bias per ionization mode against a DT reference library.
when_to_use_negative:
- Data were not spiked with U13C-labeled yeast internal standards, so no 'heavy' entries
  can be matched to the DTCCSN2 library
- The lipid class-adduct combination is not in the compatibility table (e.g., TG with
  [M+H]) or fewer than 3 U13C-labeled lipids per class-adduct combination are available,
  which restricts the downstream linear-regression CCS correction
- Measured CCS values were acquired under drift gas or calibration conditions incompatible
  with the N2-based DTCCSN2 library
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MobiLipid
  role: Automated R Markdown (MobiLipid_CCS-bias-calculation.Rmd) that performs the
    CCS bias calculation of measured U13C-labeled lipids against the DTCCSN2 library
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (version 4.3.2) for executing the bias-calculation R Markdown
    and required packages (rmarkdown, tcltk, data.table, ggplot2)
  repo: https://cran.r-project.org/
- name: RStudio
  role: Programming environment (2023.09.1) whose console is used to run the tk_choose.files
    file-selection code and rmarkdown::render call
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:cd84ccb0ec9410657af468fa469e6a37dc7068ff330538b4b261f19cc3791ad3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - RStudio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ccs-bias-calculation-ion-mobility-lipidomics

## Summary

Quantifies collision cross section (CCS) bias in IM-MS lipidomics by comparing measured TIMCCSN2 values of U13C-labeled yeast lipid internal standards against the DTCCSN2 reference library, reporting mean absolute bias per ionization mode using the MobiLipid R Markdown workflow.

## When to use

Use when LC-IM-MS lipidomics samples (e.g., from a Bruker timsTOF Pro TIM-MS or Agilent 6560 DTIM-MS) are spiked with U13C-labeled yeast extract standards and you need to quantify CCSN2 bias per ionization mode against a DT reference library.

## When NOT to use

- Data were not spiked with U13C-labeled yeast internal standards, so no 'heavy' entries can be matched to the DTCCSN2 library
- The lipid class-adduct combination is not in the compatibility table (e.g., TG with [M+H]) or fewer than 3 U13C-labeled lipids per class-adduct combination are available, which restricts the downstream linear-regression CCS correction
- Measured CCS values were acquired under drift gas or calibration conditions incompatible with the N2-based DTCCSN2 library

## Inputs

- Measured data .csv with headers 'File', 'LipidClass', 'LipidSpecies', 'Adduct', 'Label', 'CCS' (Adduct limited to [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]; Label = 'light' or 'heavy')
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values, Table S1)
- MobiLipid_CCS-bias-calculation.Rmd from github.com/FelinaHildebrand/MobiLipid

## Outputs

- CCS_bias_no_correction table: per-lipid CCS bias (%) relative to the DTCCSN2 library
- Mean absolute CCSN2 bias summarized by ionization mode (positive and negative)
- Rendered .html and .pdf report (full tables in .html; .pdf shows first 10 rows per table)
- .RData file storing all results as a list readable with readRDS()

## How to apply

In R/RStudio (R 4.3.2), launch 'MobiLipid_CCS-bias-calculation.Rmd' via rmarkdown::render and select the .Rmd, the measured-data .csv, and the 'U13C_DT_CCS_library.csv' through tcltk::tk_choose.files dialogs. The measured .csv must carry the headers 'File', 'LipidClass', 'LipidSpecies', 'Adduct', 'Label', 'CCS', with adducts restricted to [M+H], [M+NH4], [M+Na], [M-H], and [M+HCOO] and class-adduct combinations constrained by the compatibility table (e.g., PC with [M+H], [M+Na], [M+HCOO]). Restrict bias computation to rows labeled 'heavy' (U13C-labeled internal standard lipids) and valid class-adduct pairs, match each lipid species-adduct to its DTCCSN2 library entry (377 values for U13C-labeled yeast lipids), and compute per-lipid bias (%) with the library value as reference. Aggregate absolute per-lipid biases into the CCS_bias_no_correction table and summarize mean absolute bias by ionization mode. Rationale: internal standardization with a fully U13C-labeled biomass avoids the dissimilarity of external calibrants and the limited lipid count of deuterium-labeled mixtures, enabling CCS monitoring without additional external calibration measurements.

## Related tools

- **MobiLipid** (Automated R Markdown (MobiLipid_CCS-bias-calculation.Rmd) that performs the CCS bias calculation of measured U13C-labeled lipids against the DTCCSN2 library) — github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (version 4.3.2) for executing the bias-calculation R Markdown and required packages (rmarkdown, tcltk, data.table, ggplot2)) — https://cran.r-project.org/
- **RStudio** (Programming environment (2023.09.1) whose console is used to run the tk_choose.files file-selection code and rmarkdown::render call)

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation.Rmd", output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- Computed mean absolute CCSN2 bias per ionization mode reproduces the reported values of approximately 0.78% in positive and 0.33% in negative ionization mode (TIMCCSN2 vs DTCCSN2 reference)
- Bias computation includes only rows labeled 'heavy' (U13C-labeled internal standard lipids) and only class-adduct combinations valid per the compatibility table
- All Adduct values in the input .csv are among [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]
- Output includes a CCS_bias_no_correction table and both .html and .pdf reports are generated by rmarkdown::render

## Limitations

- CCS correction (downstream) requires a minimum of 3 lipids per lipid class-adduct combination, restricting correction to 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG); bias calculation itself applies only to library-covered class-adduct pairs
- No software version is specified for MobiLipid, and no repository changelog is available
- Pop-up file-selection windows may open behind the main RStudio window, and lipid/adduct nomenclature must exactly match the DTCCSN2 library

## Evidence

- [abstract] mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [intro] the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies is 0.78% in positive ionization mode and 0.33% in negative ionization mode: "the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies is 0.78% in positive ionization mode and 0.33% in negative ionization mode"
- [methods] calculate CCS bias between measured CCS values and library values of U13C labeled lipids: "calculate CCS bias between measured CCS values and library values of U<sup>13</sup>C labeled lipids"
- [readme] MobiLipid is a R markdown enabling CCS quality control for IM-MS lipidomics by internal standardization: "MobiLipid is a R markdown enabling CCS quality control for IM-MS lipidomics by internal standardization."
