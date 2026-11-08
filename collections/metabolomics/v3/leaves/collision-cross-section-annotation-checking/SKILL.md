---
name: collision-cross-section-annotation-checking
description: Use when a U13C-labeled lipid CCS library or import file must be sanity-checked
  before CCS bias calculation, e.g. verifying that U13C_DT_CCS_library.csv contains
  377 values across 162 lipid species and 15 classes.
when_to_use_negative:
- Data is a raw mzML or demultiplexed LC-DTIM-MS run that has not yet been processed
  through Skyline or IM-MS Reprocessor — CCS values do not yet exist to validate
- Input is a deuterium-labeled lipid internal standard mixture rather than the U13C-labeled
  yeast library (stable isotope-labeled lipid mixtures offer only a limited number
  of lipids)
- Goal is CCS bias calculation or correction itself, not validation of the annotation
  schema — use MobiLipid's CCS-bias-calculation Rmd instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: MobiLipid
  role: provides the DTCCSN2 reference library and the R Markdown that consumes the
    validated import CSV for CCS bias calculation
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R
  role: environment (R 4.3.2) in which the library CSV is loaded and checked with
    data.table
- name: RStudio
  role: interactive environment (2023.09.1) for running the validation and MobiLipid
    R Markdowns
- name: data.table
  role: fread() loader for U13C_DT_CCS_library.csv during count/range checks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:5aab1c305087fadc6b90be8bda4e68f68451cd12b0dba7ac635693083d7b25ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - RStudio
  - data.table
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# collision-cross-section-annotation-checking

## Summary

Validates that every collision cross section (CCS) annotation in an IM-MS lipidomics library — value counts, lipid species, class labels, adducts, and numeric ranges — matches the published MobiLipid DTCCSN2 library for U13C-labeled yeast lipids.

## When to use

Use when a U13C-labeled lipid CCS library or import file must be sanity-checked before CCS bias calculation, e.g. verifying that U13C_DT_CCS_library.csv contains 377 values across 162 lipid species and 15 classes.

## When NOT to use

- Data is a raw mzML or demultiplexed LC-DTIM-MS run that has not yet been processed through Skyline or IM-MS Reprocessor — CCS values do not yet exist to validate
- Input is a deuterium-labeled lipid internal standard mixture rather than the U13C-labeled yeast library (stable isotope-labeled lipid mixtures offer only a limited number of lipids)
- Goal is CCS bias calculation or correction itself, not validation of the annotation schema — use MobiLipid's CCS-bias-calculation Rmd instead

## Inputs

- U13C_DT_CCS_library.csv (DTCCSN2 library for U13C-labeled yeast lipids)
- Measured data .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS
- Lipid class–adduct combination table (Cer: [M+H], [M+Na], [M+HCOO]; Co: [M+NH4], [M+Na]; etc.)

## Outputs

- library_validation_report.csv with pass/fail for each check
- Validated library with 377 DTCCSN2 values, 162 lipid species, 15 classes
- Confirmed adduct annotation set ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO])

## How to apply

Load U13C_DT_CCS_library.csv from the MobiLipid GitHub repository in R (data.table::fread) and assert that it contains exactly 377 DTCCSN2 values, 162 distinct U13C-labeled lipid species, and the 15 lipid classes AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, and TG. Check every adduct annotation against the permissible set [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO] and confirm each lipid class–adduct pairing matches the combination table (e.g. Co/DG/TG only with [M+NH4] or [M+Na], SPH only with [M+H]). Verify numeric ranges: all m/z within 318.3501–966.9794 and all CCS within 189.4–327.1 Å2. For measured import data, confirm the CSV headers File, LipidClass, LipidSpecies, Adduct, Label, CCS and that Label is 'light' or 'heavy'. Write a validation report recording pass/fail per check; the library is valid only if counts, labels, adduct annotations, and both numeric ranges match exactly.

## Related tools

- **MobiLipid** (provides the DTCCSN2 reference library and the R Markdown that consumes the validated import CSV for CCS bias calculation) — https://github.com/FelinaHildebrand/MobiLipid
- **R** (environment (R 4.3.2) in which the library CSV is loaded and checked with data.table)
- **RStudio** (interactive environment (2023.09.1) for running the validation and MobiLipid R Markdowns)
- **data.table** (fread() loader for U13C_DT_CCS_library.csv during count/range checks)

## Examples

```
Rscript -e 'lib <- data.table::fread("U13C_DT_CCS_library.csv"); stopifnot(nrow(lib)==377, uniqueN(lib$LipidSpecies)==162)'; rmarkdown::render("MobiLipid_CCS-bias-calculation.Rmd")
```

## Evaluation signals

- Row count of U13C_DT_CCS_library.csv equals exactly 377 DTCCSN2 values
- Number of distinct LipidSpecies entries equals 162
- Set of LipidClass labels is exactly {AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG}
- Every Adduct value ∈ {[M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]} and each class–adduct pair matches the combination table
- All m/z ∈ [318.3501, 966.9794] and all CCS ∈ [189.4, 327.1] Å2; measured import CSV has exactly the six required headers

## Limitations

- CCS correction via linear regression requires a minimum of 3 lipids per lipid class–adduct combination, restricting correction to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG
- Not all adducts are usable for every lipid class; wrong class–adduct pairing will fail validation
- Mass error must be ≤5 ppm and CCS relative standard deviation <1% for library entries — values outside these quality criteria must not be included
- The library was generated from LC-DTIM-MS measurements of a U13C yeast extract dilution series; biases relative to TIM-MS measurements are 0.78% (positive) and 0.33% (negative) mean absolute

## Evidence

- [intro] The full DTCCSN2 library for U13C-labeled yeast lipids is provided in the Supporting Information (Table S1): "The full DTCCSN2 library for U13C-labeled
yeast lipids is provided in the Supporting Information (Table S1)"
- [methods] U13C_DT_CCS_library.csv file name: "the .csv file of the <sup>DT</sup>CCS<sub>N2</sub> library for U<sup>13</sup>C labeled lipids of yeast (“U13C_DT_CCS_library.csv”)"
- [readme] Required headers for measured data import CSV: "The .csv file has to have the following headers: “File”, “LipidClass”,
“LipidSpecies”, “Adduct”, “Label”, “CCS”"
- [readme] Permissible adducts list: "The following adducts are possible: \[M+H\],
        \[M+NH<sub>4</sub>\], \[M+Na\], \[M-H\], and \[M+HCOO\]. Not all
        adducts can be used for each lipid class."
- [abstract] Sample library composition claim surfaced by task card: "a newly established DT CCS N2 library for uniformly (U) 13 C-labeled lipids, derived from a U 13 C-labeled yeast extract, containing 377 DT CCS N2 values"
- [readme] Minimum 3 lipids per class-adduct combination for correction: "linear regression functions which require a minimum of 3 lipids
within a lipid class-adduct combination which restricts the CCS
correction to the following lipid classes: Cer, DG, HexCer, LPC, PA,"
- [intro] Library quality filter thresholds: "(1) high mass accuracy (≤5 ppm error)

(2) confirmed detection in at least two samples

(3) confirmed coelution
of adducts

(4) a CCS relative standard deviation <1%"
