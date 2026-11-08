---
name: u13c-labeled-internal-standard-ccs-qc
description: Use when LC-IM-MS lipidomics samples are spiked with U13C-labeled standards
  and you need to monitor or correct CCSN2 bias without additional external calibrant
  measurements, e.g. TIM-MS data with 0.78%/0.33% mode-dependent bias.
when_to_use_negative:
- When the lipid class-adduct combination has fewer than 3 detected U13C-labeled lipids,
  since linear correction functions cannot be built and only CCS bias calculation
  is possible
- When samples have not been spiked with U13C-labeled standards — the DTCCSN2 library
  matching and internal standardization approach cannot be applied
- For lipid classes outside the supported list (AcCa, Cer, Co, DG, HexCer, LPC, LPE,
  PA, PC, PE, PG, PI, PS, SPH, TG) or adducts other than [M+H], [M+NH4], [M+Na], [M-H],
  [M+HCOO]
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MobiLipid
  role: R Markdown tool that automates CCS bias calculation and lipid class-adduct-specific
    CCS correction using linear regression on U13C-labeled internal standards
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R / RStudio
  role: Runtime environment (R 4.3.2, RStudio 2023.09.1) required to execute the MobiLipid
    R Markdown and generate HTML/PDF reports
  repo: https://cran.r-project.org/
- name: Skyline
  role: Used to process LC-IM-MS data (LC peak integration with ECN model, IM library
    creation, ion mobility filtering) to yield the measured CCS values fed into MobiLipid
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:705f6f1fa2b1c9647f92051fb9071e514fbb85370dbda594d3f5a28717d37ce1
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
  - Skyline
  - ggbeeswarm
  - ggplot2
  - tcltk
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# u13c-labeled-internal-standard-ccs-qc

## Summary

Internal CCS quality control and correction for IM-MS lipidomics using U13C-labeled yeast lipid standards and the MobiLipid R Markdown, which computes CCS bias against a DTCCSN2 library and builds lipid class-adduct-specific linear correction functions.

## When to use

Use when LC-IM-MS lipidomics samples are spiked with U13C-labeled standards and you need to monitor or correct CCSN2 bias without additional external calibrant measurements, e.g. TIM-MS data with 0.78%/0.33% mode-dependent bias.

## When NOT to use

- When the lipid class-adduct combination has fewer than 3 detected U13C-labeled lipids, since linear correction functions cannot be built and only CCS bias calculation is possible
- When samples have not been spiked with U13C-labeled standards — the DTCCSN2 library matching and internal standardization approach cannot be applied
- For lipid classes outside the supported list (AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG) or adducts other than [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]

## Inputs

- Measured lipidomics CSV with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS
- U13C_DT_CCS_library.csv (DTCCSN2 library for U13C-labeled yeast lipids, 377 values, Table S1)
- Skyline-processed LC-IM-MS measurement results (from Agilent 6560 DTIM-MS or Bruker timsTOF Pro)
- LC-IM-MS data files spiked with U13C-labeled yeast extract internal standards

## Outputs

- CCS_bias_no_correction.csv (CCS bias % before correction)
- Correction_functions.csv (all generated CCS correction functions with slopes)
- CCS_bias_mean_by_function.csv and CCS_bias_mean_all_functions.csv
- Corrected_CCS_values.csv and Corrected_CCS_values_mean.csv
- HTML/PDF report with CCS bias tables, violin plots, and resampling summaries
- .RData list of all results

## How to apply

Spike IM-MS samples with fully U13C-labeled yeast extract standards and process LC-IM-MS data with Skyline to obtain measured CCS values, importing them as a CSV with headers File, LipidClass, LipidSpecies, Adduct, Label, and CCS (Label = light/heavy). Import U13C_DT_CCS_library.csv and compute the CCS bias (%) between measured and DTCCSN2 library values per U13C-labeled lipid. Build correction functions via linear regression (y = m*x + b, measured CCS as x, library value as y) for each lipid class-adduct combination, resampling 3–6 lipids per function to generate up to 100 functions per class; a minimum of 3 lipids restricts correction to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG. Rank the generated functions by regression slope deviation from 1 and their CCS bias distributions to identify subpar functions (e.g. slopes 0.01, 0.46, 2.42 for functions 31, 93, 98), then compare mean absolute post-correction bias for 3-lipid vs 4–6-lipid functions — higher lipid counts yield more robust correction approaching ~0%. Apply all functions to measured CCS values irrespective of labeling status, and monitor correction by re-computing bias of corrected CCS against library values.

## Related tools

- **MobiLipid** (R Markdown tool that automates CCS bias calculation and lipid class-adduct-specific CCS correction using linear regression on U13C-labeled internal standards) — https://github.com/FelinaHildebrand/MobiLipid
- **R / RStudio** (Runtime environment (R 4.3.2, RStudio 2023.09.1) required to execute the MobiLipid R Markdown and generate HTML/PDF reports) — https://cran.r-project.org/
- **Skyline** (Used to process LC-IM-MS data (LC peak integration with ECN model, IM library creation, ion mobility filtering) to yield the measured CCS values fed into MobiLipid)

## Examples

```
Rmd <- tcltk::tk_choose.files(caption = "Select R markdown (.Rmd file) which should be used for data processing:"); rmarkdown::render(paste0(Rmd), output_format = c("html_document", "pdf_document"))
```

## Evaluation signals

- Regression slopes of generated correction functions cluster near 1; outlier functions (e.g. 31, 93, 98) have slopes 0.01, 0.46, and 2.42 that should match the reported values
- Mean absolute CCS bias after correction approaches ~0% for the 10 lipid classes, down from 0.78% (positive) / 0.33% (negative) before correction
- Mean absolute post-correction bias for 4–6-lipid functions is lower than for 3-lipid functions, confirming robustness increases with more U13C-labeled lipids
- CCS relative standard deviation <1% must hold for library inclusion; measured data must pass ≤5 ppm mass error and detection in ≥2 dilutions filters
- Output CSVs exist with expected headers and the .RData list loads via readRDS() with the documented result structure

## Limitations

- CCS correction is restricted to lipid classes with at least 3 U13C-labeled lipids per class-adduct combination (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG)
- Up to 100 distinct correction functions per class are generated via resampling 3–6 lipids; functions using fewer labeled lipids are less robust
- Only the five adducts [M+H], [M+NH4], [M+Na], [M-H], and [M+HCOO] are supported, with per-class restrictions
- Nomenclature of LipidClass and LipidSpecies must exactly match the DTCCSN2 library or matching fails
- No changelog or software version is provided for MobiLipid, complicating reproducibility tracking

## Evidence

- [abstract] Internal standardization workflow definition: "we introduce MobiLipid, a novel tool facilitating the CCS quality control of IM-MS lipidomics workflows by internal standardization"
- [abstract] Library size and library basis: "a newly established DT CCS N2 library for uniformly (U) 13 C-labeled lipids, derived from a U 13 C-labeled yeast extract, containing 377 DT CCS N2 values"
- [abstract] Bias magnitudes motivating correction: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [readme] Correction function construction rules: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression. Input values for linear regression using the equation y ="
- [readme] Input data format specification: "This needs to be a .csv file containing the measured data. The .csv file has to have the following headers: "File", "LipidClass", "LipidSpecies", "Adduct", "Label", "CCS""
- [intro] Library inclusion filter thresholds: "(4) a CCS relative standard deviation <1%"
- [intro] Data processing filter criteria: "The following filter criteria were applied: (1) low mass error of ≤5 ppm, (2) detection in at least 2 dilutions"
- [results] Correction function robustness finding: [evidence span withheld — anchor status: not_found]
- [readme] More labeled lipids = more robust correction: [evidence span withheld — anchor status: not_found]
