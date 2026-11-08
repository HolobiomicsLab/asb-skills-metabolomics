---
name: csv-data-integrity-checking
description: Use when a user receives a DTCCSN2 library or measured-lipid CSV (e.g.
  U13C_DT_CCS_library.csv) and must confirm its row counts, headers, adduct labels,
  lipid-class/adduct pairings, and numeric ranges before downstream CCS bias calculation.
when_to_use_negative:
- The input is Skyline-exported or vendor IM-MS raw data (e.g. multiplexed LC-DTIM-MS
  files) rather than an already-exported .csv of CCS values — those need demultiplexing
  via PNNL PreProcessor and Skyline processing first.
- The task is CCS bias calculation or correction itself rather than pre-flight validation
  of the .csv — that is MobiLipid_CCS-bias-calculation.Rmd's job.
- The .csv has already been validated and only needs reading with fread() for downstream
  steps.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: MobiLipid
  role: The R Markdown pipeline whose required inputs (U13C_DT_CCS_library.csv and
    the measured-data CSV with specific headers) are the objects this integrity check
    validates before running.
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R (data.table)
  role: Used with data.table::fread to load the .csv library file and inspect headers,
    counts, ranges, and permissible adduct labels.
  repo: https://cran.r-project.org/
- name: RStudio
  role: The environment in which the validation checks and fread-based inspection
    are executed.
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:5e26bd188a579892f5707c3010da48f72c7df85765ac5c6398bf896879756eec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R (data.table)
  - RStudio
  - data.table
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# csv-data-integrity-checking

## Summary

Validates a DTCCSN2 lipid CCS library CSV for schema, row counts, value ranges, and permissible adduct/class annotations before it is consumed by the MobiLipid R Markdown pipeline.

## When to use

Use when a user receives a DTCCSN2 library or measured-lipid CSV (e.g. U13C_DT_CCS_library.csv) and must confirm its row counts, headers, adduct labels, lipid-class/adduct pairings, and numeric ranges before downstream CCS bias calculation.

## When NOT to use

- The input is Skyline-exported or vendor IM-MS raw data (e.g. multiplexed LC-DTIM-MS files) rather than an already-exported .csv of CCS values — those need demultiplexing via PNNL PreProcessor and Skyline processing first.
- The task is CCS bias calculation or correction itself rather than pre-flight validation of the .csv — that is MobiLipid_CCS-bias-calculation.Rmd's job.
- The .csv has already been validated and only needs reading with fread() for downstream steps.

## Inputs

- U13C_DT_CCS_library.csv — DTCCSN2 library for U13C-labeled yeast lipids from the MobiLipid GitHub repository
- Measured lipid data .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS (e.g. Example_data_import.csv)

## Outputs

- library_validation_report.csv recording pass/fail for each integrity check (value count, species count, class labels, adduct annotations, m/z and CCS ranges)
- Flagged list of rows with non-permissible adducts or out-of-range m/z/CCS values

## How to apply

Load the .csv in R with data.table::fread (the MobiLipid README names 'U13C_DT_CCS_library.csv' as the required library file). For a measured-data import, verify the headers are exactly File, LipidClass, LipidSpecies, Adduct, Label, CCS; for the DTCCSN2 library, verify the 377 DTCCSN2 values span 162 distinct U13C-labeled yeast lipid species across 15 lipid classes (AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG). Check every Adduct annotation against the permissible set [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] and confirm each lipid class-adduct pairing matches the documented class-adduct combination table (e.g. AcCa only [M+H]/[M-H], TG only [M+NH4]/[M+Na]). Confirm m/z values lie within 318.3501-966.9794 and CCS values within 189.4-327.1 A2, and that Label uses only 'light' or 'heavy'. Emit a validation report marking pass/fail per check so that library-dependent steps (CCS bias calculation and linear-regression correction functions requiring ≥3 lipids per class-adduct combination) are not run on malformed data.

## Related tools

- **MobiLipid** (The R Markdown pipeline whose required inputs (U13C_DT_CCS_library.csv and the measured-data CSV with specific headers) are the objects this integrity check validates before running.) — https://github.com/FelinaHildebrand/MobiLipid
- **R (data.table)** (Used with data.table::fread to load the .csv library file and inspect headers, counts, ranges, and permissible adduct labels.) — https://cran.r-project.org/
- **RStudio** (The environment in which the validation checks and fread-based inspection are executed.) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
data.table::fread("U13C_DT_CCS_library.csv") |> check_headers(c("File","LipidClass","LipidSpecies","Adduct","Label","CCS")) |> check_ranges(mz = c(318.3501, 966.9794), ccs = c(189.4, 327.1))
```

## Evaluation signals

- Row count of the library file equals 377 DTCCSN2 values and 162 distinct U13C-labeled lipid species across exactly 15 lipid class labels.
- Every Adduct annotation is one of [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]; no other adduct strings appear.
- Each lipid class-adduct pairing is in the documented class-adduct combination table (e.g. AcCa:[M+H],[M-H]; TG:[M+NH4],[M+Na]); cross-check against the README table.
- All m/z values fall within 318.3501-966.9794 and all CCS values fall within 189.4-327.1 A2.
- Measured-data CSV headers match exactly File, LipidClass, LipidSpecies, Adduct, Label, CCS and Label contains only 'light' or 'heavy'.

## Limitations

- The 377-value/162-species/15-class counts are specific to this U13C yeast extract library; other DTCCSN2 libraries would need their own expected counts and ranges.
- MobiLipid's CCS correction requires at least 3 lipids per lipid class-adduct combination, so a library or measurement set that is schema-valid but sparse per combination still cannot support correction for classes other than Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG.
- The class-adduct combination table is fixed in the README; custom adducts or lipid classes outside that table cannot be validated against it without editing MobiLipid itself.

## Evidence

- [methods] library validation report: "This needs to be a .csv file containing the measured data. The .csv file has to have the following headers: "File", "LipidClass", "LipidSpecies", "Adduct", "Label", "CCS""
- [readme] adduct and class-adduct table: "The following adducts are possible: \[M+H\], \[M+NH<sub>4</sub>\], \[M+Na\], \[M-H\], and \[M+HCOO\]. Not all adducts can be used for each lipid class."
- [abstract] library size and class counts: "a newly established DT CCS N2 library for uniformly (U) 13 C-labeled lipids, derived from a U 13 C-labeled yeast extract, containing 377 DT CCS N2 values"
- [intro] numeric ranges: [evidence span withheld — anchor status: not_found]
- [readme] minimum lipids per class-adduct combination: "linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination which restricts the CCS correction to the following lipid classes: Cer, DG, HexCer, LPC, PA,"
- [methods] fread loading instruction: "the .csv file of the <sup>DT</sup>CCS<sub>N2</sub> library for U<sup>13</sup>C labeled lipids of yeast (“U13C_DT_CCS_library.csv”)"
