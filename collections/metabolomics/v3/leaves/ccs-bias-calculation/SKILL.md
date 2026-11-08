---
name: ccs-bias-calculation
description: Use when measured (LC-)IM-MS lipidomics data contain U13C-labeled ('heavy')
  lipid entries with CCS values and you need to quantify CCSN2 bias against the DTCCSN2
  library, per lipid class-adduct combination.
when_to_use_negative:
- You only have CCS values without any U13C-labeled ('heavy') lipid entries — the
  DTCCSN2 library only contains values for uniformly 13C-labeled yeast lipids, so
  nothing can be matched.
- Your goal is CCS correction rather than bias quantification alone — use MobiLipid_CCS-bias-calculation_CCS-correction.Rmd,
  which additionally builds linear correction functions requiring a minimum of 3 lipids
  per lipid class-adduct combination.
- Your measured data use adducts outside [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] or
  lipid class-adduct combinations absent from the library's permitted table.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: R Markdown tool that computes percent CCS bias between measured CCS values
    and DTCCSN2 library values of U13C-labeled lipids
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R Markdown / rmarkdown
  role: Renders the MobiLipid_CCS-bias-calculation.Rmd workflow to html/pdf output
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R 4.3.2) for data filtering, evaluation, and visualization
  repo: https://cran.r-project.org/
- name: RStudio
  role: IDE (RStudio 2023.09.1) in which the MobiLipid markdown is run via the console
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:4ea58bf67e86f541d9a307cb52b074ec3f3469ace8a2396ff4fea52c6bf785e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R Markdown
  - R Markdown / rmarkdown
  - RStudio
  - ggbeeswarm
  - ggplot2
  - tcltk
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# ccs-bias-calculation

## Summary

Calculates the percent CCS bias between measured IM-MS CCS values and DTCCSN2 library values of U13C-labeled yeast lipids using MobiLipid, enabling internal standardization-based CCS quality control of IM-MS lipidomics without external calibration measurements.

## When to use

Use when measured (LC-)IM-MS lipidomics data contain U13C-labeled ('heavy') lipid entries with CCS values and you need to quantify CCSN2 bias against the DTCCSN2 library, per lipid class-adduct combination.

## When NOT to use

- You only have CCS values without any U13C-labeled ('heavy') lipid entries — the DTCCSN2 library only contains values for uniformly 13C-labeled yeast lipids, so nothing can be matched.
- Your goal is CCS correction rather than bias quantification alone — use MobiLipid_CCS-bias-calculation_CCS-correction.Rmd, which additionally builds linear correction functions requiring a minimum of 3 lipids per lipid class-adduct combination.
- Your measured data use adducts outside [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] or lipid class-adduct combinations absent from the library's permitted table.

## Inputs

- Measured lipid CCS data as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS (e.g. Example_data_import.csv)
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values, Table S1)

## Outputs

- CCS_bias_no_correction.csv — table of percent CCS bias before correction for each matched lipid
- MobiLipid_CCS-bias-calculation_<filename>.html and .pdf report with CCS bias tables and bias distribution plots
- .RData file storing all results as a list readable with readRDS()

## How to apply

In R/RStudio, run MobiLipid_CCS-bias-calculation.Rmd via rmarkdown::render with tcltk file choosers, supplying (1) the measured data .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS (Label = 'light'/'heavy') and (2) U13C_DT_CCS_library.csv (377 DTCCSN2 values for U13C-labeled yeast lipids). Restrict entries to valid lipid class-adduct combinations (e.g. PC with [M+H], [M+Na], [M+HCOO]; TG with [M+NH4], [M+Na]; adducts limited to [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]) and ensure nomenclature matches the library. Match heavy lipids to library entries on LipidClass-LipidSpecies-Adduct and compute percent CCS bias as the relative deviation of measured CCS (x) from library DTCCSN2 value (y); aggregate to mean absolute bias per ionization mode. For unlabeled lipids, use matched monoisotopic ('light') entries against the same library values, expecting a small systematic ~0.25% DTCCSN2 shift for U13C-labeled vs unlabeled pairs. Rationale: calibrants often have chemical properties dissimilar to analytes, so a fully U13C-labeled biomass provides a matched internal standard palette across 10 lipid classes.

## Related tools

- **MobiLipid** (R Markdown tool that computes percent CCS bias between measured CCS values and DTCCSN2 library values of U13C-labeled lipids) — github.com/FelinaHildebrand/MobiLipid
- **R Markdown / rmarkdown** (Renders the MobiLipid_CCS-bias-calculation.Rmd workflow to html/pdf output) — github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R 4.3.2) for data filtering, evaluation, and visualization) — https://cran.r-project.org/
- **RStudio** (IDE (RStudio 2023.09.1) in which the MobiLipid markdown is run via the console) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
Rmd <- tcltk::tk_choose.files(caption='Select R markdown'); rmarkdown::render(Rmd, output_format=c('html_document','pdf_document'))
```

## Evaluation signals

- CCS_bias_no_correction.csv contains a percent bias entry for every matched lipid class-species-adduct triple.
- For unlabeled lipids vs the TIM reference, mean absolute CCSN2 bias reproduces ~0.73% (positive) and ~0.46% (negative) ionization mode; for U13C-labeled lipids ~0.78% (positive) and ~0.33% (negative).
- Matched light/heavy lipid pairs show a systematic DTCCSN2 shift of approximately 0.25% (Supporting Information Figure S5).
- Mean absolute biases aggregate to approximately 0% across 10 lipid classes only after applying MobiLipid's CCS correction — pre-correction values should be nonzero and class-specific.
- Rendered html report includes per-lipid CCS bias table and bias distribution figure for each lipid class-adduct combination.

## Limitations

- The DTCCSN2 library covers 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG for correction; bias calculation additionally supports AcCa, Co, LPE, PG, SPH) and only the adducts [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO].
- A U13C-labeled vs unlabeled systematic DTCCSN2 shift of ~0.25% exists due to reduced mass considerations observed with moderate resolution IM-MS.
- MobiLipid has no published software version or repository changelog, complicating reproducibility pinning.

## Evidence

- [methods] calculate CCS bias between measured CCS values and library values of U13C labeled lipids: "calculate CCS bias between measured CCS values and library values of U<sup>13</sup>C labeled lipids"
- [readme] README running MobiLipid: [evidence span withheld — anchor status: not_found]
- [readme] README output description: "CCS_bias_no_correction: Table with CCS bias (%) before correction"
- [readme] README input header requirement: "The .csv file has to have the following headers: "File", "LipidClass", "LipidSpecies", "Adduct", "Label", "CCS""
- [abstract] abstract bias magnitude: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
