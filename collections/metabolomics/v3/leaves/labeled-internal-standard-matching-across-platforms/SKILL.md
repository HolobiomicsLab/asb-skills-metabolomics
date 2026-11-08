---
name: labeled-internal-standard-matching-across-platforms
description: Use when LC-IM-MS data contain U13C-labeled internal standards and you
  need to quantify CCSN2 bias of measured CCS values against a DTCCSN2 reference library
  across ion mobility technologies or ionization modes.
when_to_use_negative:
- The data are unlabeled lipids only with no U13C-labeled internal standards present,
  since matching relies on 'heavy' Label entries spiked as fully U13C-labeled yeast
  extract (Neubauer et al. 2012)
- The measured lipids use adducts or lipid class nomenclature outside the permitted
  MobiLipid combinations (e.g. AcCa with [M+Na], SPH with [M-H]) or with fewer than
  3 heavy lipids per class-adduct combination, which blocks CCS correction
- The goal is IM platform calibration itself rather than post-acquisition CCS bias
  assessment; MobiLipid assumes vendor-specific CCS calibration (e.g. IM-MS Browser
  10.0) has already been performed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: R Markdown tool that matches heavy-lipid measurements to the U13C_DT_CCS_library.csv
    and computes per-lipid CCS bias (optionally correction functions per class-adduct
    combination)
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R version 4.3.2) in which the MobiLipid R Markdown is
    rendered; required with packages htmltools, rmarkdown, data.table, ggplot2, DT,
    webshot, tcltk, knitr, ggbeeswarm
  repo: https://cran.r-project.org/
- name: RStudio
  role: IDE used to install packages, run the tcltk file-picker and rmarkdown::render
    workflow, and inspect exported .csv/.RData results
  repo: https://www.rstudio.com/products/rstudio/download/
- name: Skyline
  role: Upstream LC-IM-MS processing (version 23.1.0.268) that yields the lipid CCS
    values feeding the import .csv; performs LC peak integration, IM library creation,
    and IM filtering
- name: IM-MS Browser
  role: Performs the single-field CCS calibration (version 10.0) that must precede
    CCS bias matching on the DTIM-MS reference data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:fb56feb2a9d5336dd5029d7250b375ce023c100e5ede4b74759119de3eeb8fbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - IM-MS Browser
  - MobiLipid
  - R
  - R Markdown
  - RStudio
  - Skyline
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# labeled-internal-standard-matching-across-platforms

## Summary

Matches U13C-labeled ('heavy') internal-standard lipids measured on one IM platform (e.g. TIM-MS) to the U13C_DT_CCS_library.csv DTCCSN2 reference values from a DTIM-MS library, computing per-lipid percent CCS bias by lipid class-adduct combination with MobiLipid.

## When to use

Use when LC-IM-MS data contain U13C-labeled internal standards and you need to quantify CCSN2 bias of measured CCS values against a DTCCSN2 reference library across ion mobility technologies or ionization modes.

## When NOT to use

- The data are unlabeled lipids only with no U13C-labeled internal standards present, since matching relies on 'heavy' Label entries spiked as fully U13C-labeled yeast extract (Neubauer et al. 2012)
- The measured lipids use adducts or lipid class nomenclature outside the permitted MobiLipid combinations (e.g. AcCa with [M+Na], SPH with [M-H]) or with fewer than 3 heavy lipids per class-adduct combination, which blocks CCS correction
- The goal is IM platform calibration itself rather than post-acquisition CCS bias assessment; MobiLipid assumes vendor-specific CCS calibration (e.g. IM-MS Browser 10.0) has already been performed

## Inputs

- Measured LC-IM-MS lipid CCS data as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS (e.g. Example_data_import.csv), Label = 'heavy' for U13C-labeled lipids
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 DTCCSN2 values) shipped with MobiLipid

## Outputs

- CCS_bias_no_correction.csv — per-lipid CCS bias (%) before correction
- HTML/PDF report with CCS bias tables, beeswarm/violin plots, and resampling summaries (e.g. MobiLipid_CCS-bias-calculation_<datafile>.html)
- .RData list file storing all results, readable with readRDS()
- For the correction variant: Correction_functions.csv, CCS_bias_mean_by_function.csv, CCS_bias_mean_all_functions.csv, Corrected_CCS_values.csv, Corrected_CCS_values_mean.csv

## How to apply

In R/RStudio with the installed MobiLipid packages, import the measured lipid CCS data as a .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS, where Label is 'heavy' for U13C-labeled lipids and 'light' for natural lipids. Import the provided U13C_DT_CCS_library.csv and match 'heavy' lipid entries to library entries using identical lipid class and lipid species nomenclature and the same adduct, restricting to the permitted lipid class-adduct combinations (e.g. PC [M+H]/[M+Na]/[M+HCOO], TG [M+NH4]/[M+Na], PE [M+H]/[M+Na]/[M-H]). Compute the percent CCS bias of each matched lipid as the relative deviation between the measured CCS value (x) and the DTCCSN2 library value (y), i.e. measured CCS vs library reference. Run 'MobiLipid_CCS-bias-calculation.Rmd' for bias calculation alone, or 'MobiLipid_CCS-bias-calculation_CCS-correction.Rmd' to additionally build lipid class- and adduct-specific linear correction functions (y = m*x + b) requiring at least 3 heavy lipids per class-adduct combination. Inspect the resulting bias table, violin plots, and the exported CCS_bias_no_correction.csv, and judge the match by whether the mean absolute CCSN2 bias falls within the ~0.3-0.8% range reported for TIM-MS vs DTCCSN2 (0.78% positive, 0.33% negative for U13C-labeled lipids).

## Related tools

- **MobiLipid** (R Markdown tool that matches heavy-lipid measurements to the U13C_DT_CCS_library.csv and computes per-lipid CCS bias (optionally correction functions per class-adduct combination)) — https://github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R version 4.3.2) in which the MobiLipid R Markdown is rendered; required with packages htmltools, rmarkdown, data.table, ggplot2, DT, webshot, tcltk, knitr, ggbeeswarm) — https://cran.r-project.org/
- **RStudio** (IDE used to install packages, run the tcltk file-picker and rmarkdown::render workflow, and inspect exported .csv/.RData results) — https://www.rstudio.com/products/rstudio/download/
- **Skyline** (Upstream LC-IM-MS processing (version 23.1.0.268) that yields the lipid CCS values feeding the import .csv; performs LC peak integration, IM library creation, and IM filtering)
- **IM-MS Browser** (Performs the single-field CCS calibration (version 10.0) that must precede CCS bias matching on the DTIM-MS reference data)

## Examples

```
Rmd <- tcltk::tk_choose.files(caption = "Select R markdown (.Rmd file) which should be used for data processing:", multi = FALSE); data_import <- tcltk::tk_choose.files(caption = "Data import (measured data as CSV file):", multi = FALSE); rmarkdown::render(paste0(Rmd), output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- Every matched pair shares identical LipidClass, LipidSpecies, and Adduct strings between the import .csv and U13C_DT_CCS_library.csv, and the import .csv passes the header check (File, LipidClass, LipidSpecies, Adduct, Label, CCS)
- All matched entries use adducts from the permitted lipid class-adduct table (e.g. PC [M+H]/[M+Na]/[M+HCOO], PA [M+NH4]/[M+Na]/[M-H])
- Mean absolute CCSN2 bias per ionization mode lands in the expected ~0.3-0.8% window (0.78% positive, 0.33% negative for U13C-labeled lipids on TIM-MS vs DTCCSN2)
- If correction is enabled, each used correction function is a linear regression y = m*x + b built from 3-6 heavy lipids, and corrected mean absolute biases across the 10 lipid classes drop to approximately 0%
- Output artifacts exist as .html/.pdf plus CCS_bias_no_correction.csv, with the .RData list loadable via readRDS() and full tables visible only in the .html (PDF shows only first 10 rows per table)

## Limitations

- CCS correction is restricted to 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) and requires a minimum of 3 U13C-labeled lipids per lipid class-adduct combination
- Matching requires the same lipid class and lipid species nomenclature as the DTCCSN2 library; inconsistent naming breaks the match
- The DTCCSN2 library covers U13C-labeled yeast lipids only — 377 values — so bias assessment is limited to lipids representable by that biomass extract
- MobiLipid performs no CCS calibration itself; it assumes prior vendor-specific calibration (e.g. IM-MS Browser 10.0 single-field calibration) has already been done
- Correction functions are built via resampling (up to 100 distinct functions using 3-6 lipids), so corrected bias estimates vary by function and should be examined as distributions, not single values

## Evidence

- [abstract] Matching internal-standard lipids across platforms: "utilizing a newly established DTCCSN2 library for U13C-labeled lipids to internally assess CCSN2 bias in IM-MS lipidomics workflows"
- [readme] MobiLipid internal standardization: [evidence span withheld — anchor status: not_found]
- [abstract] Library size and provenance: "a newly established DT CCS N2 library for uniformly (U) 13 C-labeled lipids, derived from a U 13 C-labeled yeast extract, containing 377 DT CCS N2 values"
- [intro] Reference bias magnitude on TIM-MS: "the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies is 0.78% in positive ionization mode and 0.33% in negative ionization mode"
- [readme] Minimum lipids per class for regression: "linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination"
