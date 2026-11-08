---
name: lipid-ccs-reference-library-validation
description: Use when building or auditing a DTCCSN2 library of U13C-labeled lipids
  for internal standardization in LC-IM-MS lipidomics, or when verifying that a supplied
  library (e.g. U13C_DT_CCS_library.csv) meets stated quality criteria.
when_to_use_negative:
- Input data come from a TIM-MS (e.g. Bruker timsTOF Pro) or other non-DTIM platform
  — the library must be built from DTCCSN2 (Agilent 6560 drift-tube) measurements,
  not from trapped-IM values.
- The analytes are unlabeled ('light') lipids only — the library and validation apply
  specifically to U13C-labeled ('heavy') internal standards spiked into samples.
- A lipid class-adduct combination appears in the data but is not listed as permissible
  (e.g. LPC with [M-H]) — such entries must be rejected rather than added to the library.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MobiLipid
  role: R Markdown pipeline that consumes the validated DTCCSN2 library for CCS bias
    calculation and correction; the library validation step gates whether the library
    is fit for these downstream computations.
  repo: github.com/FelinaHildebrand/MobiLipid
- name: Skyline 23.1.0.268
  role: Processes LC-IM-MS data to produce the per-lipid measured CCS values and adduct/lipid-class
    annotations that are filtered and curated into the library.
- name: PNNL PreProcessor 4.12
  role: Demultiplexes and smooths multiplexed LC-DTIM-MS acquisitions before Skyline
    processing and library curation.
- name: IM-MS Browser 10.0
  role: Performs single-field CCS calibration within the Agilent 6560 DTIM-MS workflow
    that yields the DTCCSN2 reference values.
- name: R / RStudio 4.3.2 / 2023.09.1
  role: Environment for the data filtering, evaluation, and visualization steps (including
    the ECN-model trend plot and validation report generation).
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:ed182e51dbf395227587bc7b1fe4ec9ae2195ac276bc39b762cc5874c0f49e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IM-MS Browser 10.0
  - MobiLipid
  - PNNL PreProcessor 4.12
  - R
  - R / RStudio 4.3.2 / 2023.09.1
  - RStudio
  - Skyline 23.1.0.268
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# lipid-ccs-reference-library-validation

## Summary

Curation and validation of a DTCCSN2 reference library for U13C-labeled yeast lipids, generated from Agilent 6560 DTIM-MS measurements and used inside MobiLipid for internal CCS bias calculation and correction.

## When to use

Use when building or auditing a DTCCSN2 library of U13C-labeled lipids for internal standardization in LC-IM-MS lipidomics, or when verifying that a supplied library (e.g. U13C_DT_CCS_library.csv) meets stated quality criteria.

## When NOT to use

- Input data come from a TIM-MS (e.g. Bruker timsTOF Pro) or other non-DTIM platform — the library must be built from DTCCSN2 (Agilent 6560 drift-tube) measurements, not from trapped-IM values.
- The analytes are unlabeled ('light') lipids only — the library and validation apply specifically to U13C-labeled ('heavy') internal standards spiked into samples.
- A lipid class-adduct combination appears in the data but is not listed as permissible (e.g. LPC with [M-H]) — such entries must be rejected rather than added to the library.

## Inputs

- LC-DTIM-MS raw data of a U13C-labeled yeast extract dilution series (positive and negative ionization modes)
- Skyline-processed IM-MS export with File, LipidClass, LipidSpecies, Adduct, Label, CCS columns
- U13C_DT_CCS_library.csv (the DTCCSN2 library candidate)
- Lipid class-adduct combination table (permissible pairings per lipid class)

## Outputs

- Curated DTCCSN2 library with 377 values for 162 U13C-labeled lipid species across 15 lipid classes
- Library validation report (per-entry pass/fail against the four filter criteria and adduct-combination rules)
- Diagnostic plot of DTCCSN2 entries per lipid class versus m/z (ECN-model analogy check)

## How to apply

Generate the library from LC-DTIM-MS measurements of a yeast extract dilution series on the Agilent 6560 DTIM-MS in positive and negative ionization modes, then apply the article's four inclusion filters: mass error ≤5 ppm, detection in at least 2 dilutions, coelution of adducts (including matching of positive and negative ionization mode), and a CCS relative standard deviation <1% for repeatability precision. Verify library values by plotting entries per lipid class versus m/z using the equivalent carbon number (ECN) model analogy, which reveals whether entries follow expected class-specific CCS-vs-m/z trends. Confirm each lipid class-adduct pairing matches the permissible adduct set ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]) tabulated in the README, e.g. AcCa accepts only [M+H]/[M-H] while TG accepts only [M+NH4]/[M+Na]. Record the final library size (377 DTCCSN2 values covering 162 U13C-labeled species across 15 lipid classes: AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG) and write a validation report flagging any entry that fails a filter or an adduct-combination rule.

## Related tools

- **MobiLipid** (R Markdown pipeline that consumes the validated DTCCSN2 library for CCS bias calculation and correction; the library validation step gates whether the library is fit for these downstream computations.) — github.com/FelinaHildebrand/MobiLipid
- **Skyline 23.1.0.268** (Processes LC-IM-MS data to produce the per-lipid measured CCS values and adduct/lipid-class annotations that are filtered and curated into the library.)
- **PNNL PreProcessor 4.12** (Demultiplexes and smooths multiplexed LC-DTIM-MS acquisitions before Skyline processing and library curation.)
- **IM-MS Browser 10.0** (Performs single-field CCS calibration within the Agilent 6560 DTIM-MS workflow that yields the DTCCSN2 reference values.)
- **R / RStudio 4.3.2 / 2023.09.1** (Environment for the data filtering, evaluation, and visualization steps (including the ECN-model trend plot and validation report generation).) — https://cran.r-project.org/

## Examples

```
Rscript -e 'lib <- data.table::fread("U13C_DT_CCS_library.csv"); stopifnot(nrow(lib)==377, uniqueN(lib$LipidSpecies)==162, uniqueN(lib$LipidClass)==15)'
```

## Evaluation signals

- Library contains exactly 377 DTCCSN2 values for 162 distinct U13C-labeled lipid species spanning 15 lipid classes (AcCa, Cer, Co, DG, HexCer, LPC, LPE, PA, PC, PE, PG, PI, PS, SPH, TG).
- Every entry survives all four filters: mass error ≤5 ppm, detection in ≥2 dilutions, adduct coelution with positive/negative mode matching, and CCS RSD <1%.
- No lipid class-adduct pairing violates the README table (e.g. Co/DG/TG must appear only as [M+NH4] or [M+Na]; SPH only as [M+H]).
- Plotting library entries per lipid class versus m/z shows the expected ECN-model-consistent trend with no outlier entries.
- Validation report records pass/fail per criterion and the library is accepted only when all checks pass.

## Limitations

- CCS correction downstream of the library is restricted to the 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) that support linear regression with ≥3 lipids per class-adduct combination.
- Adduct coverage is limited to five species ([M+H], [M+NH4], [M+Na], [M-H], [M+HCOO]); adducts outside this set cannot be validated or corrected.
- Library quality depends on upstream demultiplexing (PNNL PreProcessor) and single-field CCS calibration (IM-MS Browser) being performed correctly; errors there propagate into DTCCSN2 values.
- No software changelog or explicit version for MobiLipid is documented in the repository, complicating reproducibility audits.

## Evidence

- [intro] Library inclusion filter: mass error ≤5 ppm: "(1) high mass accuracy (≤5 ppm error)"
- [intro] Library inclusion filter: confirmed detection in at least two samples: "(2) confirmed detection in at least two samples"
- [intro] Library inclusion filter: confirmed coelution of adducts: "(3) confirmed coelution
of adducts"
- [intro] Library inclusion filter: CCS relative standard deviation <1%: "(4) a CCS relative standard deviation <1%"
- [intro] Generate the DTCCSN2 library from LC-DTIM-MS measurements of a yeast extract dilution series in positive and negative ionization modes: "Specifically, LC-DTIM-MS measurements of a yeast
extract dilution series in positive and negative ionization modes
served for library generation"
- [intro] Verify library DTCCSN2 values by plotting entries per lipid class versus m/z using the ECN-model analogy: "the resulting DTCCSN2 values were verified by
plotting the entries for each lipid class versus the respective m/z values"
- [abstract] MobiLipid requires only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes: "requiring only three U13C-labeled lipids per lipid class-adduct combination across 10 lipid classes without requiring additional external measurements"
- [readme]  permissible adduct set: "The following adducts are possible: \[M+H\], \[M+NH<sub>4</sub>\], \[M+Na\], \[M-H\], and \[M+HCOO\]. Not all
        adducts can be used for each lipid class."
- [readme]  input schema: "The .csv file has to have the following headers: "File", "LipidClass",
    "LipidSpecies", "Adduct", "Label", "CCS""
- [readme]  library file name: "It is provided with the code
    and called "*U13C_DT_CCS_library.csv*"."
