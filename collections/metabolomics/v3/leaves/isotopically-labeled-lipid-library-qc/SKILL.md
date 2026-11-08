---
name: isotopically-labeled-lipid-library-qc
description: Use when a workflow imports U13C_DT_CCS_library.csv from the FelinaHildebrand/MobiLipid
  repository and must confirm the 377-value, 162-lipid-species, 15-class curation
  is intact before internal standardization.
when_to_use_negative:
- Input is the measured .csv export from Skyline or IM-MS processing (columns File,
  LipidClass, LipidSpecies, Adduct, Label, CCS) rather than the curated DTCCSN2 library
  — use MobiLipid's bias-calculation Rmd instead.
- The library already passed validation in a prior session and has not changed on
  disk — re-running adds no information.
- The analysis targets non-labeled (light) analyte CCS correction functions themselves
  rather than validating the reference library contents.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: Supplies the curated U13C DTCCSN2 library being validated and defines the
    expected class-adduct combinations
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R (data.table)
  role: Loads U13C_DT_CCS_library.csv and performs the count, label, and range checks
  repo: https://cran.r-project.org/
- name: RStudio
  role: Environment in which the validation R script is run and the report is generated
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:ade24affffd8984402b9ce7a1a6518642e0f1b28f278716ce61e653a09e15b9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - R (data.table)
  - RStudio
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# isotopically-labeled-lipid-library-qc

## Summary

Validate the MobiLipid DTCCSN2 library for U13C-labeled yeast lipids against the published counts, lipid-class labels, adduct annotations, and m/z/CCS numeric ranges before using it for CCS bias calculation or correction.

## When to use

Use when a workflow imports U13C_DT_CCS_library.csv from the FelinaHildebrand/MobiLipid repository and must confirm the 377-value, 162-lipid-species, 15-class curation is intact before internal standardization.

## When NOT to use

- Input is the measured .csv export from Skyline or IM-MS processing (columns File, LipidClass, LipidSpecies, Adduct, Label, CCS) rather than the curated DTCCSN2 library — use MobiLipid's bias-calculation Rmd instead.
- The library already passed validation in a prior session and has not changed on disk — re-running adds no information.
- The analysis targets non-labeled (light) analyte CCS correction functions themselves rather than validating the reference library contents.

## Inputs

- U13C_DT_CCS_library.csv (the DTCCSN2 library for U13C-labeled yeast lipids from the MobiLipid GitHub repository)
- Expected class-adduct combination table from the MobiLipid README (15 lipid classes mapped to permissible adducts)

## Outputs

- library_validation_report.csv recording pass/fail for value count, species count, class labels, adduct annotations, and m/z and CCS numeric ranges

## How to apply

Load the library file in R with data.table::fread and confirm it holds exactly 377 DTCCSN2 values covering 162 distinct U13C-labeled lipid species across 15 lipid classes (AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG). Check that every adduct annotation is one of [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] and that each class-adduct pairing matches the combination table from the README (e.g. TG only with [M+NH4]/[M+Na], SPH only with [M+H]). Verify all m/z values fall within 318.3501–966.9794 and all DTCCSN2 values within 189.4–327.1 Å², which were established by curation with mass error ≤5 ppm, detection in ≥2 dilutions, coelution of adducts, and CCS relative standard deviation <1%. Write the per-check results into a library_validation_report.csv; the library passes only if every count, label set, and numeric range matches exactly, otherwise downstream MobiLipid bias calculation or correction should not proceed.

## Related tools

- **MobiLipid** (Supplies the curated U13C DTCCSN2 library being validated and defines the expected class-adduct combinations) — github.com/FelinaHildebrand/MobiLipid
- **R (data.table)** (Loads U13C_DT_CCS_library.csv and performs the count, label, and range checks) — https://cran.r-project.org/
- **RStudio** (Environment in which the validation R script is run and the report is generated) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
lib <- data.table::fread("U13C_DT_CCS_library.csv"); stopifnot(nrow(lib) == 377, length(unique(lib$LipidSpecies)) == 162)
```

## Evaluation signals

- dtccsn2 value count equals exactly 377 and distinct lipid species count equals exactly 162
- Lipid class labels collapse to exactly 15 classes: AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG
- Every adduct string is in {[M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]} and each class-adduct pair appears in the README combination table
- All m/z values are within 318.3501–966.9794 and all CCS values within 189.4–327.1 Å²
- library_validation_report.csv exists and records pass for every check

## Limitations

- The library only covers the 15 lipid classes in the file; classes outside this set cannot be validated or corrected with MobiLipid.
- CCS correction via MobiLipid requires ≥3 lipids per class-adduct combination, so some class-adduct pairs in the library may never support correction.
- Library curation quality depends on the original filtering (≤5 ppm mass error, ≥2 dilutions, adduct coelution, CCS RSD <1%); validation cannot detect errors that passed those thresholds.

## Evidence

- [results] library contains 377 DTCCSN2 values for 162 U13C-labeled yeast lipids: [evidence span withheld — anchor status: not_found]
- [results] 15 lipid classes in the library: [evidence span withheld — anchor status: not_found]
- [readme] permissible adduct set: "The following adducts are possible: \[M+H\], \[M+NH<sub>4</sub>\], \[M+Na\], \[M-H\], and \[M+HCOO\]"
- [intro] curation quality criteria for library entries: [evidence span withheld — anchor status: not_found]
- [readme] library file name and location: "the .csv file of the <sup>DT</sup>CCS<sub>N2</sub> library for U<sup>13</sup>C labeled lipids of yeast (“U13C_DT_CCS_library.csv”)"
