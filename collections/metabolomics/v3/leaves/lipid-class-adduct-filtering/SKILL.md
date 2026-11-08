---
name: lipid-class-adduct-filtering
description: Use when a measured IM-MS lipidomics table (File, LipidClass, LipidSpecies,
  Adduct, Label, CCS) must be matched against the U13C_DT_CCS_library.csv DTCCSN2
  library and invalid or nomenclature-mismatched class-adduct pairs must be excluded
  before CCS bias calculation.
when_to_use_negative:
- The input table lacks CCS values or uses adducts outside the five supported types
  ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]) — such rows are out of MobiLipid's scope
  and must not be force-matched
- LipidClass or LipidSpecies nomenclature differs from the DTCCSN2 library and cannot
  be harmonized — string matching will silently drop or mismatch entries
- The analysis goal is general lipid annotation or quantification without CCS quality
  control — this skill only gates class-adduct validity for CCS bias/correction
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MobiLipid
  role: R Markdown workflow whose lipid class-adduct combination table defines the
    filter; runs CCS bias calculation on the filtered matches
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R / RStudio
  role: Environment for loading the .csv inputs, applying the class-adduct filter,
    and rendering MobiLipid_CCS-bias-calculation.Rmd
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:c0b1c252dde5efb6f93cfa356acd38f8a1bddc55e3bf64e1b9164a1f7193e6f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R / RStudio
  - R Markdown
  - RStudio
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lipid-class-adduct-filtering

## Summary

Restrict IM-MS lipidomics CCS analysis to the valid lipid class-adduct combinations supported by MobiLipid's DTCCSN2 library (e.g., PC with [M+H]/[M+Na]/[M+HCOO]; TG with [M+NH4]/[M+Na]), ensuring measured lipids are matched to library entries only when class, species, and adduct nomenclature agree.

## When to use

Use when a measured IM-MS lipidomics table (File, LipidClass, LipidSpecies, Adduct, Label, CCS) must be matched against the U13C_DT_CCS_library.csv DTCCSN2 library and invalid or nomenclature-mismatched class-adduct pairs must be excluded before CCS bias calculation.

## When NOT to use

- The input table lacks CCS values or uses adducts outside the five supported types ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]) — such rows are out of MobiLipid's scope and must not be force-matched
- LipidClass or LipidSpecies nomenclature differs from the DTCCSN2 library and cannot be harmonized — string matching will silently drop or mismatch entries
- The analysis goal is general lipid annotation or quantification without CCS quality control — this skill only gates class-adduct validity for CCS bias/correction

## Inputs

- Measured IM-MS lipidomics data as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS (e.g., Example_data_import.csv)
- DTCCSN2 library for U13C-labeled yeast lipids as U13C_DT_CCS_library.csv (377 values)

## Outputs

- Filtered set of valid lipid class-adduct matched light/heavy entries for CCS bias calculation
- CCS_bias_no_correction.csv with percent CCS bias per matched lipid

## How to apply

Parse the measured .csv, which must carry exactly the headers File, LipidClass, LipidSpecies, Adduct, Label, and CCS, and normalize LipidClass/LipidSpecies/Adduct strings to the same nomenclature used in U13C_DT_CCS_library.csv. Apply the lipid class-adduct combination table as an inclusion filter (e.g., AcCa: [M+H],[M-H]; Cer: [M+H],[M+Na],[M+HCOO]; DG: [M+NH4],[M+Na]; PA: [M+NH4],[M+Na],[M-H]; PC: [M+H],[M+Na],[M+HCOO]; PE: [M+H],[M+Na],[M-H]; PI/PS/PG/LPC/HexCer per table; TG: [M+NH4],[M+Na]; SPH: [M+H]), discarding rows with adducts outside the allowed set. Use the Label column ('light' vs 'heavy') to separate monoisotopic from U13C-labeled entries and match pairs on LipidClass-LipidSpecies-Adduct. Note that CCS correction functions additionally require at least 3 lipids per class-adduct combination (MobiLipid computes up to 100 functions using 3 to 6 lipids), so combinations with fewer than 3 matched heavy lipids support bias calculation but not correction.

## Related tools

- **MobiLipid** (R Markdown workflow whose lipid class-adduct combination table defines the filter; runs CCS bias calculation on the filtered matches) — github.com/FelinaHildebrand/MobiLipid
- **R / RStudio** (Environment for loading the .csv inputs, applying the class-adduct filter, and rendering MobiLipid_CCS-bias-calculation.Rmd) — https://cran.r-project.org/

## Examples

```
Rmd <- tcltk::tk_choose.files(filters=matrix(c("Rmd files","Rmd"),1,2)); data_import <- tcltk::tk_choose.files(filters=matrix(c("csv files","csv"),1,2)); rmarkdown::render(Rmd, output_format=c("html_document","pdf_document"))
```

## Evaluation signals

- Every retained row has an adduct from {M+H, M+NH4, M+Na, M-H, M+HCOO} that is permitted for its lipid class per the MobiLipid combination table
- Each filtered heavy entry has a corresponding library entry matched on LipidClass-LipidSpecies-Adduct with identical nomenclature
- CCS_bias_no_correction.csv contains a percent bias entry for each matched lipid (as required by the task validation)
- For the labeled/unlabeled DTIM-MS comparison, the aggregated mean bias reproduces the ~0.25% systematic DTCCSN2 shift reported in Supporting Information Figure S5
- Class-adduct combinations kept for CCS correction have at least 3 U13C-labeled lipids (3–6 used per regression function)

## Limitations

- Only 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) support CCS correction, and only with the listed adducts; other combinations can be bias-calculated but not corrected
- The adduct set is fixed to five ion types; adducts like [M+K] or [M+CH3COO] are not handled
- Linear regression correction requires a minimum of 3 lipids within a lipid class-adduct combination; fewer matched heavy lipids restrict the workflow to bias calculation

## Evidence

- [readme] The .csv file has to have the following headers: “File”, “LipidClass”, “LipidSpecies”, “Adduct”, “Label”, “CCS”: "The .csv file has to have the following headers: “File”, “LipidClass”, “LipidSpecies”, “Adduct”, “Label”, “CCS”"
- [readme] adduct enumeration and class restriction: "The following adducts are possible: \[M+H\], \[M+NH<sub>4</sub>\], \[M+Na\], \[M-H\], and \[M+HCOO\]. Not all adducts can be used for each lipid class."
- [readme] minimum lipids for correction functions: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression."
- [methods] CCS bias calculation step: "calculate CCS bias between measured CCS values and library values of U<sup>13</sup>C labeled lipids"
