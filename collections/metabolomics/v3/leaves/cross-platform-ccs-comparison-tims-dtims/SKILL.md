---
name: cross-platform-ccs-comparison-tims-dtims
description: Use when IM-MS lipidomics data from two ion-mobility technologies (trapped
  IM and drift-tube IM) must be compared, e.g., to compute mean absolute CCSN2 bias
  per ionization mode or to decide whether class-adduct-specific CCS correction is
  needed.
when_to_use_negative:
- Input CCS data are from a single IM technology with no DTIM reference and no U13C-labeled
  internal standards spiked in — internal standardization cannot be performed
- Lipid classes fall outside the supported list (AcCa, Cer, Co, DG, HexCer, LPC, LPE,
  PA, PC, PE, PG, PI, PS, SPH, TG) or use adducts not permitted for that class
- Goal is absolute CCS prediction rather than bias assessment/correction against a
  DTCCSN2 library
edam_operation: http://edamontology.org/operation_3861
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: Automated R Markdown performing CCS-bias calculation and lipid class/adduct-specific
    correction between measured TIM-MS values and the DTCCSN2 library
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Programming environment in which MobiLipid runs; required installation alongside
    RStudio
  repo: https://cran.r-project.org/
- name: Skyline
  role: Used upstream to process LC-IM-MS data (peak integration via ECN model, IM
    library creation, IM filtering) producing the measured CCS values fed into the
    comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:aaaba39bc431c1b27656f86b5ed2270fe089a76eb1d1e28f403d92a770e40c3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R Markdown
  - Skyline
  edam_operation: http://edamontology.org/operation_3861
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# cross-platform-ccs-comparison-tims-dtims

## Summary

Quantify and interpret CCSN2 bias between TIM-MS (e.g., timsTOF Pro) and DTIM-MS (Agilent 6560) lipid measurements using a U13C-labeled yeast lipid DTCCSN2 library, enabling cross-platform CCS quality control and class-specific correction in MobiLipid.

## When to use

Use when IM-MS lipidomics data from two ion-mobility technologies (trapped IM and drift-tube IM) must be compared, e.g., to compute mean absolute CCSN2 bias per ionization mode or to decide whether class-adduct-specific CCS correction is needed.

## When NOT to use

- Input CCS data are from a single IM technology with no DTIM reference and no U13C-labeled internal standards spiked in — internal standardization cannot be performed
- Lipid classes fall outside the supported list (AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG) or use adducts not permitted for that class
- Goal is absolute CCS prediction rather than bias assessment/correction against a DTCCSN2 library

## Inputs

- .csv of measured lipid CCS values with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS
- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids (377 values, also Table S1)
- IM-MS data processed via Skyline (LC peak integration with ECN model, IM library, IM filtering)

## Outputs

- CCS_bias_no_correction.csv — per-lipid CCS bias (%) table
- Per-mode mean absolute CCSN2 bias (e.g., 0.73% positive, 0.46% negative for unlabeled lipids)
- .html/.pdf report with bias tables, resampling summaries, violin plots before/after correction
- Correction_functions.csv, Corrected_CCS_values.csv, CCS_bias_mean_by_function.csv and related CSVs when the correction workflow is run
- .RData list of all results readable via readRDS()

## How to apply

Match measured lipid entries (heavy/light) to the DTCCSN2 library (U13C_DT_CCS_library.csv) using identical lipid class, species, and adduct nomenclature, restricting to permitted adducts per class (e.g., TG only [M+NH4]/[M+Na]; PC [M+H]/[M+Na]/[M+HCOO]). Feed measured CCS values (x) and library DTCCSN2 values (y) into MobiLipid's CCS-bias-calculation routine to compute percent CCS bias per lipid, then aggregate absolute biases per ionization mode. For correction, MobiLipid computes up to 100 distinct linear correction functions (y = m*x + b) per lipid class-adduct combination by resampling 3–6 U13C-labeled lipids, which requires ≥3 labeled lipids detected in that class-adduct combination. Correct all measured CCS values irrespective of labeling status and re-evaluate bias against the library. Verify by comparing computed mean absolute bias to the article's reported ranges (e.g., 0.73% positive / 0.46% negative for unlabeled lipids; 0.78% / 0.33% for labeled) and inspecting per-ion-species distributions for strong species-specific trends.

## Related tools

- **MobiLipid** (Automated R Markdown performing CCS-bias calculation and lipid class/adduct-specific correction between measured TIM-MS values and the DTCCSN2 library) — github.com/FelinaHildebrand/MobiLipid
- **R** (Programming environment in which MobiLipid runs; required installation alongside RStudio) — https://cran.r-project.org/
- **Skyline** (Used upstream to process LC-IM-MS data (peak integration via ECN model, IM library creation, IM filtering) producing the measured CCS values fed into the comparison)

## Examples

```
rmarkdown::render(Rmd, output_format = c("html_document", "pdf_document"), output_file = paste0(Rmd_name,"_",filename,".html"))  # Rmd = MobiLipid_CCS-bias-calculation.Rmd; data import = measured CCS .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS; library = U13C_DT_CCS_library.csv
```

## Evaluation signals

- Mean absolute CCSN2 bias for unlabeled lipids lands near reported values: 0.73% (positive mode) and 0.46% (negative mode); 0.78%/0.33% for U13C-labeled lipids
- After CCS correction, mean absolute biases across 10 lipid classes are reduced to approximately 0%
- Every lipid class-adduct combination uses only adducts permitted in the README table (e.g., TG limited to [M+NH4]/[M+Na])
- Linear correction functions are only generated for class-adduct combinations with ≥3 U13C-labeled lipids detected
- Output CSV headers match the required schema (File, LipidClass, LipidSpecies, Adduct, Label, CCS) and CCS_bias_no_correction.csv is produced

## Limitations

- Requires samples spiked with fully U13C-labeled yeast extract (Neubauer et al. 2012) internal standards; not applicable to unspiked data
- CCS correction is restricted to 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) needing ≥3 labeled lipids per class-adduct combination
- Tables in the .pdf output show only the first 10 rows; full tables must be read from the .html output
- No software version or changelog is provided for MobiLipid
- Tool output 'no strong ion species-specific trends' is observational; per-species trends were not quantified as significant

## Evidence

- [abstract] Mean absolute bias reported for labeled lipids: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [abstract] Correction reduces bias to ~0%: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [abstract] Library size: "a newly established DT CCS N2 library for uniformly (U) 13 C-labeled lipids, derived from a U 13 C-labeled yeast extract, containing 377 DT CCS N2 values"
- [intro] Uncorrected bias between technologies: "the mean absolute CCSN2 bias of U13C-labeled lipids between the two IM technologies
is 0.78% in positive ionization mode and 0.33% in negative ionization
mode"
- [abstract] Three U13C lipids per class-adduct requirement: "requiring only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes without requiring additional external measurements"
- [intro] R Markdown tool nature: "CCS monitoring and correction is automatized in form of an R Markdown."
- [readme] README invocation block: "# Choose R markdown which should be processed
Rmd <- tcltk::tk_choose.files(caption = "Select R markdown (.Rmd file) which should be used for data processing:"
