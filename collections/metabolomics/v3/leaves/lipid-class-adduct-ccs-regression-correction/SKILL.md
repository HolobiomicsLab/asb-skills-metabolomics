---
name: lipid-class-adduct-ccs-regression-correction
description: Use when LC-IM-MS lipidomics samples are spiked with fully U13C-labeled
  yeast extract lipids, measured CCS values exist per lipid class/adduct, and TIMCCSN2-vs-DTCCSN2
  bias needs correction via lipid class- and adduct-specific linear regression.
when_to_use_negative:
- The goal is only to monitor CCS bias without correction - use MobiLipid_CCS-bias-calculation.Rmd
  instead of the CCS-correction Rmd.
- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids - linear regression
  requires a minimum of 3 lipids, so correction is restricted to Cer, DG, HexCer,
  LPC, PA, PC, PE, PI, PS, and TG (AcCa, Co, LPE, PG, SPH are not correctable).
- The data are from unlabeled-only workflows without U13C yeast extract spike-in -
  the DTCCSN2 library values for U13C-labeled lipids are the regression targets and
  cannot be substituted.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: R Markdown tool that resamples 3 to 6 U13C-labeled lipids per lipid class-adduct
    combination, fits up to 100 linear correction functions, applies them to all measured
    CCS values, and reports pre/post-correction CCS bias
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Programming environment (version 4.3.2) in which the correction and bias aggregation
    run; dependencies include data.table, ggplot2, DT, webshot, tcltk, knitr, ggbeeswarm,
    rmarkdown, htmltools
  repo: https://cran.r-project.org/
- name: RStudio
  role: IDE where the MobiLipid R Markdown is rendered and console code is executed
    to choose files and run rmarkdown::render
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:f5e21897c449803dfd14dbc31ffa73716aa6f5a3e8ede42f76c0296419f1d185
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R Markdown
  - RStudio
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# lipid-class-adduct-ccs-regression-correction

## Summary

A MobiLipid procedure that computes lipid class- and adduct-specific linear CCS correction functions (y = m*x + b) by regressing measured CCS of U13C-labeled lipids against their DTCCSN2 library values, then applies those functions to all measured CCS values regardless of labeling status, driving mean absolute CCSN2 bias toward 0%.

## When to use

Use when LC-IM-MS lipidomics samples are spiked with fully U13C-labeled yeast extract lipids, measured CCS values exist per lipid class/adduct, and TIMCCSN2-vs-DTCCSN2 bias needs correction via lipid class- and adduct-specific linear regression.

## When NOT to use

- The goal is only to monitor CCS bias without correction - use MobiLipid_CCS-bias-calculation.Rmd instead of the CCS-correction Rmd.
- A lipid class-adduct combination has fewer than 3 U13C-labeled lipids - linear regression requires a minimum of 3 lipids, so correction is restricted to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG (AcCa, Co, LPE, PG, SPH are not correctable).
- The data are from unlabeled-only workflows without U13C yeast extract spike-in - the DTCCSN2 library values for U13C-labeled lipids are the regression targets and cannot be substituted.

## Inputs

- Measured LC-IM-MS lipid data as a .csv with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS; supported adducts are [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]
- DTCCSN2 library for U13C-labeled yeast lipids (U13C_DT_CCS_library.csv, 377 DTCCSN2 values, Table S1)
- MobiLipid R Markdown 'MobiLipid_CCS-bias-calculation_CCS-correction.Rmd'

## Outputs

- Corrected_CCS_values.csv: corrected CCS values and CCS bias after correction for each lipid and each correction function
- Corrected_CCS_values_mean.csv: mean corrected CCS values for each lipid over all correction functions with the same number of labeled lipids used
- Correction_functions.csv: all generated CCS correction functions (linear m and b per lipid class-adduct combination)
- CCS_bias_mean_by_function.csv: mean CCS bias per lipid class-adduct combination for each correction function
- CCS_bias_mean_all_functions.csv: mean CCS bias per combination over all correction functions with the same number of lipids
- CCS_bias_no_correction.csv and a .RData list of all results
- Rendered HTML/PDF report (rmarkdown::render) with bias tables, resampling summary, and violin plots of bias before/after correction

## How to apply

Load the measured data .csv (headers File, LipidClass, LipidSpecies, Adduct, Label, CCS) and the U13C_DT_CCS_library.csv into R using data.table. For each lipid class-adduct combination (e.g. PC [M+H], DG [M+NH4]), restrict to heavy (U13C-labeled) lipids and fit linear correction functions y = m*x + b, using measured CCS as x and DTCCSN2 library values as y; MobiLipid resamples 3 to 6 labeled lipids per function and generates up to 100 distinct functions per combination. Apply every generated function to all measured CCS values of that combination, irrespective of labeling status, producing Corrected_CCS_values. Validate by recomputing the CCS bias (%) of corrected values against the DTCCSN2 library for U13C-labeled lipids and aggregating the mean bias per correction function and across all functions sharing the same number of labeled lipids used to generate them. Correct application is confirmed when mean absolute CCSN2 bias per lipid class drops toward approximately 0%, matching the paper's reduction from 0.78% (positive) and 0.33% (negative) pre-correction.

## Related tools

- **MobiLipid** (R Markdown tool that resamples 3 to 6 U13C-labeled lipids per lipid class-adduct combination, fits up to 100 linear correction functions, applies them to all measured CCS values, and reports pre/post-correction CCS bias) — github.com/FelinaHildebrand/MobiLipid
- **R** (Programming environment (version 4.3.2) in which the correction and bias aggregation run; dependencies include data.table, ggplot2, DT, webshot, tcltk, knitr, ggbeeswarm, rmarkdown, htmltools) — https://cran.r-project.org/
- **RStudio** (IDE where the MobiLipid R Markdown is rendered and console code is executed to choose files and run rmarkdown::render) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
Rmd <- tcltk::tk_choose.files(caption = "Select R markdown (.Rmd file) which should be used for data processing:", multi = FALSE); rmarkdown::render(Rmd, output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- Mean absolute CCSN2 bias per lipid class-adduct combination drops to approximately 0% after correction, versus pre-correction 0.78% (positive mode) and 0.33% (negative mode)
- CCS_bias_mean_by_function.csv and CCS_bias_mean_all_functions.csv contain entries for every lipid class-adduct combination with >=3 heavy lipids and no NA bias rows for those combinations
- Corrected_CCS_values.csv includes one corrected CCS per lipid per generated function, and CCS_bias after correction is computed only against DTCCSN2 library values of U13C-labeled lipids
- Number of generated correction functions per combination matches the resampling summary (3 to 6 lipids per function, up to 100 functions)
- Rendered HTML/PDF contains, for each combination, mean bias before/after correction grouped by number of U13C-labeled lipids used, plus violin plots of bias before and after correction

## Limitations

- CCS correction requires a minimum of 3 U13C-labeled lipids within a lipid class-adduct combination, restricting correction to 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG); other combinations are reported as not correctable.
- Pre-correction CCS biases between trapped IM (TIM-MS) and drift-tube IM differ by ionization mode (0.78% positive, 0.33% negative), so corrected values are validated against the DTCCSN2 reference rather than absolute accuracy.
- Corrected CCS values are generated per correction function; variability across up to 100 functions with 3-6 lipids each must be inspected via the mean tables and violin plots rather than taken as a single value.

## Evidence

- [readme] MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression."
- [readme] Input values for linear regression using the equation y = m*x + b: [evidence span withheld — anchor status: not_found]
- [readme] After generation of all CCS correction functions, all measured CCS values are corrected, irrespective of their labeling status: "After generation of all CCS correction functions, all measured CCS values are corrected, irrespective of their labeling status."
- [abstract] mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [abstract] the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [readme] Corrected_CCS_values: Table with corrected CCS values and CCS bias after correction for each lipid and each correction function: "Corrected_CCS_values: Table with corrected CCS values and CCS bias after correction for each lipid and each correction function"
