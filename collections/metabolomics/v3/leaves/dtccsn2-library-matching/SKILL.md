---
name: dtccsn2-library-matching
description: Use when IM-MS lipidomics samples are spiked with U13C-labeled yeast
  extract and measured CCSN2 values (e.g., TIM-MS) must be bias-assessed or corrected
  against reference DTCCSN2 values per lipid class-adduct combination.
when_to_use_negative:
- Samples are not spiked with U13C-labeled internal standards (fully labeled yeast
  extract), since the matching relies on 'heavy' entries with library counterparts
- The lipid class-adduct combination is outside the permitted table or fewer than
  3 U13C-labeled lipids are detected, so no linear correction function can be fitted
- The input lacks measured CCS values (no ion mobility dimension) — use conventional
  quantification/QC workflows instead
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: R Markdown tool that performs CCS bias calculation against the DTCCSN2 library
    and lipid class-adduct-specific CCS correction
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R 4.3.2) for rendering the MobiLipid R Markdown and data
    filtering with data.table
  repo: https://cran.r-project.org/
- name: RStudio 2023.09.1
  role: IDE used to run MobiLipid, with further data filtering, evaluation, and visualization
- name: Skyline 23.1.0.268
  role: Upstream processing producing the measured CCS values (LC peak integration,
    IM library creation, IM filtering) that are matched to the DTCCSN2 library
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:f5625414ef5df125708f50e6179fa483600784f03e00c7826c69d4e99ba3a6b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R Markdown
  - RStudio 2023.09.1
  - Skyline 23.1.0.268
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# dtccsn2-library-matching

## Summary

Matching measured ion-mobility CCS values of U13C-labeled yeast lipids against the MobiLipid DTCCSN2 library (377 values) to compute lipid class- and adduct-specific CCS bias and derive linear correction functions, eliminating the need for external CCS calibrant measurements.

## When to use

Use when IM-MS lipidomics samples are spiked with U13C-labeled yeast extract and measured CCSN2 values (e.g., TIM-MS) must be bias-assessed or corrected against reference DTCCSN2 values per lipid class-adduct combination.

## When NOT to use

- Samples are not spiked with U13C-labeled internal standards (fully labeled yeast extract), since the matching relies on 'heavy' entries with library counterparts
- The lipid class-adduct combination is outside the permitted table or fewer than 3 U13C-labeled lipids are detected, so no linear correction function can be fitted
- The input lacks measured CCS values (no ion mobility dimension) — use conventional quantification/QC workflows instead

## Inputs

- Measured IM-MS lipidomics data as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS (Label = 'light'/'heavy')
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values, Table S1)
- Allowed lipid class-adduct combination table (e.g., Cer: [M+H], [M+Na], [M+HCOO]; PS: [M+H], [M+Na], [M-H])

## Outputs

- CCS_bias_no_correction — table of CCS bias (%) before correction
- Correction_functions.csv — table of all fitted linear correction functions (slope m, intercept b, class-adduct combination, constituent lipids)
- CCS_bias_mean_by_function / CCS_bias_mean_all_functions — mean bias per and across correction functions
- Corrected_CCS_values / Corrected_CCS_values_mean — corrected CCS values and residual bias per lipid
- Rendered .html/.pdf report and .RData list of all results

## How to apply

Import the measured data as a .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS alongside U13C_DT_CCS_library.csv, ensuring lipid class/adduct nomenclature matches the library. Filter to the 'heavy' (U13C-labeled) entries and to permitted class-adduct combinations (e.g., PC with [M+H], [M+Na], [M+HCOO]; TG with [M+NH4], [M+Na]). Calculate the percent CCS bias between measured CCSN2 values and DTCCSN2 library values per lipid class-adduct combination. For correction, repeatedly draw 3–6 labeled lipids (minimum 3) per combination — up to 100 distinct resamples — and fit y = m*x + b with measured CCS as x and DTCCSN2 library values as y. Apply the resulting functions to correct all measured CCS values irrespective of labeling status, then recompute bias of corrected heavy-lipid CCS values against the library to verify the correction.

## Related tools

- **MobiLipid** (R Markdown tool that performs CCS bias calculation against the DTCCSN2 library and lipid class-adduct-specific CCS correction) — https://github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R 4.3.2) for rendering the MobiLipid R Markdown and data filtering with data.table) — https://cran.r-project.org/
- **RStudio 2023.09.1** (IDE used to run MobiLipid, with further data filtering, evaluation, and visualization)
- **Skyline 23.1.0.268** (Upstream processing producing the measured CCS values (LC peak integration, IM library creation, IM filtering) that are matched to the DTCCSN2 library)

## Examples

```
Rmd <- tcltk::tk_choose.files(); rmarkdown::render(Rmd, output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- Pre-correction mean absolute CCSN2 bias of heavy lipids vs the DTCCSN2 library should be in the range of ~0.78% (positive mode) and ~0.33% (negative mode) as reported for TIM vs DTIM measurements
- Post-correction mean absolute CCSN2 bias for the 10 correctable lipid classes should be reduced to approximately 0%
- Each fitted correction function must contain at least 3 (and at most 6) U13C-labeled lipids per class-adduct combination, with up to 100 distinct resamples recorded in Correction_functions.csv
- Input .csv must parse with exactly the headers File, LipidClass, LipidSpecies, Adduct, Label, CCS and only adducts from the permitted class-adduct table
- Output includes all six expected .csv tables (CCS_bias_no_correction through Corrected_CCS_values_mean) plus the rendered .html/.pdf and .RData results list

## Limitations

- CCS correction is restricted to the 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) and only class-adduct combinations with a minimum of 3 U13C-labeled lipids
- Only five adducts are supported ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]) and not all are valid for every lipid class
- No changelog or software version is specified for the MobiLipid repository, so reproducibility across versions is not guaranteed

## Evidence

- [readme] first calculate CCS bias between measured CCS values and library values: "to first calculate CCS bias between measured CCS values and library values of U<sup>13</sup>C labeled lipids."
- [readme] Resampling-based linear correction functions: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression."
- [readme] Required input schema: "The .csv file has to have the following headers: "File", "LipidClass", "LipidSpecies", "Adduct", "Label", "CCS""
- [intro] Expected bias magnitude between IM technologies: "the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies is 0.78% in positive ionization mode and 0.33% in negative ionization mode"
- [abstract] Correction outcome: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [methods] Library CCS precision criterion: "(4) a relative standard deviation <1% for CCS repeatability precision"
