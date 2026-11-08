---
name: feature-lipid-pair-tabulation
description: Use when you have a peak-picked feature table (m/z, retention time, intensities,
  e.g. from MZmine) and need to enumerate which library lipid species/adducts could
  explain each feature's precursor m/z before MS/MS validation.
when_to_use_negative:
- You already have an identification table from LipidMatch or another tool (Greazy,
  LipidSearch, MS-DIAL) and want to append/merge annotations rather than re-derive
  MS1 candidates
- You need final lipid identifications — MS1 exact mass matching alone yields precursor-level
  candidates only; LipidMatch assigns identification levels 1-4 based on observed
  MS/MS fragments
- Your data are Waters files, which LipidMatch does not currently support
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidMatch
  role: R-based tool (gWidgets/tcltk dialogue-box GUI) that performs the MS1 exact
    mass matching between feature m/z and in silico precursor ions and writes the
    match table
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MZmine
  role: Peak-picking software producing the feature table CSV (m/z, retention time,
    intensities) that serves as input to the MS1 matching step
- name: R
  role: Language in which LipidMatch is implemented and in which the matching workflow
    is run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:05aeafe80d2050d0b30f68442a853d2c87f18d43b1672acbe0e770cb5f8b09af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MZmine
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# feature-lipid-pair-tabulation

## Summary

Builds the MS1-level feature-to-candidate lipid match table used by LipidMatch: each feature's m/z from a peak-picked feature table is exact-mass matched against in silico precursor ions (lipid species + adducts) within a user-defined Da tolerance, producing the candidate list consumed by downstream MS/MS fragment matching.

## When to use

Use when you have a peak-picked feature table (m/z, retention time, intensities, e.g. from MZmine) and need to enumerate which library lipid species/adducts could explain each feature's precursor m/z before MS/MS validation.

## When NOT to use

- You already have an identification table from LipidMatch or another tool (Greazy, LipidSearch, MS-DIAL) and want to append/merge annotations rather than re-derive MS1 candidates
- You need final lipid identifications — MS1 exact mass matching alone yields precursor-level candidates only; LipidMatch assigns identification levels 1-4 based on observed MS/MS fragments
- Your data are Waters files, which LipidMatch does not currently support

## Inputs

- Feature table CSV from peak-picking software (MZmine, XCMS, or MS-DIAL) with per-feature m/z, retention time, and intensities
- LipidMatch in silico precursor-ion libraries in .csv format (56 lipid types, >250,000 lipid species, with adducts for the given polarity)

## Outputs

- Feature-candidate match table (CSV) with each feature-lipid pair, lipid name, adduct, and library precursor exact mass
- Candidate lipid lists per feature for downstream MS/MS fragment-matching steps

## How to apply

Load the feature table CSV (each feature's m/z, retention time, and intensities, e.g. exported from MZmine, XCMS, or MS-DIAL) and the LipidMatch in silico precursor-ion libraries in .csv format covering the 56 lipid types and their adducts for the chosen polarity. For each feature, perform exact mass matching at the MS1 level between in silico precursor ions and the feature's m/z using the user-defined m/z tolerance in Da (Step 1; Fig. 3), excluding dimers, multimers, and in-source fragments from matching. Record every feature-lipid pair with the lipid name, adduct, and library precursor exact mass in the match table. Save the feature-candidate match table as a CSV so it feeds Step 2 (finding MS/MS scans within RT/m/z tolerance) and the fragment-matching steps; this MS1 step only generates candidates, so do not treat matches as identifications — LipidMatch requires observed fragments for annotation.

## Related tools

- **LipidMatch** (R-based tool (gWidgets/tcltk dialogue-box GUI) that performs the MS1 exact mass matching between feature m/z and in silico precursor ions and writes the match table) — https://github.com/GarrettLab-UF/LipidMatch
- **MZmine** (Peak-picking software producing the feature table CSV (m/z, retention time, intensities) that serves as input to the MS1 matching step)
- **R** (Language in which LipidMatch is implemented and in which the matching workflow is run)

## Evaluation signals

- Every reported feature-lipid pair has a precursor exact-mass difference within the user-defined m/z tolerance in Da
- Dimers, multimers, and in-source fragments are excluded from the match table
- Each match row records lipid name, adduct, and library precursor exact mass
- The match table is saved as a CSV and is consumable by the subsequent MS/MS scan-assignment step (Step 2 of Fig. 3)
- Number of candidates per feature is consistent with the library size for the given polarity (56 lipid types, >250,000 lipid species)

## Limitations

- MS1 matching alone is ambiguous — features often contain multiple co-eluting molecules with similar m/z values, so candidates require MS/MS fragment confirmation
- LipidMatch was tested on Windows 7 through 10 and pinned to R version 3.3.3; cross-platform support is not stated
- Customization requires users to develop their own fragmentation libraries in Excel
- Key artifacts (software zip, lipid libraries in .csv) are provided only as additional files, not archived in a public repository

## Evidence

- [methods] MS1 exact mass matching: "LipidMatch performs exact mass matching at the MS1 level between in silico precursor ions and each features m/z using a user defined m/z tolerance (Da) (Step 1; Fig. 3)"
- [methods] Feature table creation: [evidence span withheld — anchor status: not_found]
- [readme] README peak-picking integration: "LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer), and combine results from other lipidomics software."
- [abstract] Library scope: "LipidMatch currently has over 250,000 lipid species spanning 56 lipid types contained in in silico fragmentation libraries."
