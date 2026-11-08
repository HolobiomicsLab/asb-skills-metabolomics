---
name: lipid-class-adduct-subset-resampling
description: Use when measured IM-MS lipidomics data (.csv with File, LipidClass,
  LipidSpecies, Adduct, Label, CCS columns) contains heavy (U13C-labeled) lipids and
  CCSN2 values need class- and adduct-specific correction against a DTCCSN2 reference
  library.
when_to_use_negative:
- Fewer than 3 U13C-labeled lipids are detected for a lipid class-adduct combination
  — the linear regression cannot be fitted and CCS correction is not possible (bias
  calculation alone still works)
- The lipid class is outside the 10 correctable classes (e.g. AcCa, Co, LPE, PG, SPH,
  Cer? — only Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG are correctable); run CCS
  bias calculation without correction instead
- The goal is absolute CCS calibration from external calibrants rather than internal
  monitoring/compensation of relative TIM vs DT CCSN2 bias
tools:
- name: MobiLipid
  role: R Markdown tool that performs the subset resampling, linear regression fitting,
    and CCS correction per class-adduct combination
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: runtime environment (R 4.3.2) in which the MobiLipid R Markdown is rendered
  repo: https://cran.r-project.org/
- name: data.table
  role: package used to load and filter the measured .csv data and the U13C_DT_CCS_library.csv
    reference file
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:8de98e5f405a286c9257d7513e536bb9c56ccb47e7711caaee30bf467d71585d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R Markdown
  - data.table
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lipid-class-adduct-subset-resampling

## Summary

Generates up to 100 linear CCS correction functions per lipid class–adduct combination by repeatedly resampling 3–6 U13C-labeled lipids and regressing measured TIMCCSN2 values against DTCCSN2 library values, enabling class-specific compensation of CCSN2 bias in IM-MS lipidomics.

## When to use

Use when measured IM-MS lipidomics data (.csv with File, LipidClass, LipidSpecies, Adduct, Label, CCS columns) contains heavy (U13C-labeled) lipids and CCSN2 values need class- and adduct-specific correction against a DTCCSN2 reference library.

## When NOT to use

- Fewer than 3 U13C-labeled lipids are detected for a lipid class-adduct combination — the linear regression cannot be fitted and CCS correction is not possible (bias calculation alone still works)
- The lipid class is outside the 10 correctable classes (e.g. AcCa, Co, LPE, PG, SPH, Cer? — only Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG are correctable); run CCS bias calculation without correction instead
- The goal is absolute CCS calibration from external calibrants rather than internal monitoring/compensation of relative TIM vs DT CCSN2 bias

## Inputs

- Measured IM-MS lipidomics data as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values, also Table S1)
- Permitted lipid class-adduct combination table (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG with adducts [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO])

## Outputs

- Correction_functions.csv — table of all fitted slopes (m), intercepts (b), class-adduct combinations, and constituent lipid sets
- Corrected_CCS_values / Corrected_CCS_values_mean tables with corrected CCS values and post-correction bias per lipid and per function
- CCS_bias_mean_by_function and CCS_bias_mean_all_functions summary tables (html/pdf report and .RData list also generated)

## How to apply

Load the measured data and U13C_DT_CCS_library.csv with data.table in R, filter to 'heavy' labeled lipids, and restrict to the 10 correctable classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) using only permitted class–adduct combinations (e.g. TG only with [M+NH4] or [M+Na]). For each combination, draw distinct subsets of 3 to 6 labeled lipids (minimum 3 required; a linear regression y = m*x + b is not possible with fewer), computing up to 100 distinct resamples per combination. For each subset, fit a linear regression with measured CCS values of U13C-labeled lipids as x and DTCCSN2 library values as y, collecting slope m, intercept b, and constituent lipid sets into Correction_functions.csv. Rationale: lipid class-adduct specificity captures technology-dependent CCS bias (e.g. TIM vs DT mean absolute biases of 0.78% positive / 0.33% negative), and resampling 3–6 lipids varies the calibration basis without requiring external measurements.

## Related tools

- **MobiLipid** (R Markdown tool that performs the subset resampling, linear regression fitting, and CCS correction per class-adduct combination) — github.com/FelinaHildebrand/MobiLipid
- **R** (runtime environment (R 4.3.2) in which the MobiLipid R Markdown is rendered) — https://cran.r-project.org/
- **data.table** (package used to load and filter the measured .csv data and the U13C_DT_CCS_library.csv reference file)

## Examples

```
rmarkdown::render("MobiLipid_CCS-bias-calculation_CCS-correction.Rmd", output_format = c("html_document", "pdf_document"), output_file = paste0("MobiLipid_CCS-bias-calculation_CCS-correction_", filename, ".html"))
```

## Evaluation signals

- Correction_functions.csv contains at most 100 distinct regression functions per lipid class-adduct combination, each built from exactly 3–6 distinct labeled lipids
- No function is generated for a class-adduct combination with fewer than 3 detected heavy lipids; the output reports 'Number of lipids detected ... and if CCS correction was possible'
- Validation: applying the functions must reduce mean absolute CCSN2 bias of labeled lipids toward approximately 0% per lipid class (from a pre-correction baseline of ~0.78% positive / 0.33% negative mode)
- Post-correction bias tables (CCS_bias_mean_by_function, CCS_bias_mean_all_functions) and violin plots show reduced CCS bias after correction for each number of lipids used in the regression
- Slope/intercept pairs vary across resamples of the same combination, confirming distinct subsets rather than a single pooled regression

## Limitations

- CCS correction is restricted to lipid classes with at least 3 U13C-labeled lipids per class-adduct combination (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG)
- Corrected CCS values are applied to all lipids irrespective of labeling status, so accuracy depends on the assumption that class-adduct linear correction transfers from labeled to unlabeled species
- No software version or changelog is provided for MobiLipid in the repository

## Evidence

- [readme] Resampling range and regression setup: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression."
- [readme] Regression input definition: [evidence span withheld — anchor status: not_found]
- [readme] Minimum lipid requirement and correctable classes: "linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination which restricts the CCS correction to the following lipid classes: Cer, DG, HexCer, LPC, PA,"
- [abstract] Post-correction validation target: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [results] Pre-correction bias magnitude: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [abstract] No external calibration required: "requiring only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes without requiring additional external measurements"
