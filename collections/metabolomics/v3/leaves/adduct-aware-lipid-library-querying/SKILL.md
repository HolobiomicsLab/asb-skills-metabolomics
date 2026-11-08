---
name: adduct-aware-lipid-library-querying
description: Use when starting lipid annotation from an MZmine/MS-DIAL/XCMS feature
  table (m/z, RT, intensities) and you need candidate lipid species/adducts matched
  by precursor exact mass before any MS/MS fragment matching.
when_to_use_negative:
- Input is still raw vendor MS data — feature detection with MZmine/XCMS/MS-DIAL must
  precede this MS1 matching step
- Only MS/MS spectral data (.ms2) are available without a peak-picked feature table
  of precursor m/z values
- Acquisition is on Waters files or lipid classes outside the library coverage (e.g.
  LipidMatch does not currently support Waters files)
tools:
- name: LipidMatch
  role: R-based tool whose in silico precursor-ion libraries and MS1 exact mass matching
    (Step 1, Fig. 3) produce the feature-candidate matches
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MZmine
  role: Peak-picking software generating the feature table (m/z, RT, intensities)
    that is queried against the lipid libraries
- name: R
  role: Language in which LipidMatch is implemented (R 3.3.3, gWidgets/tcltk interface)
    and run to execute the library query
  repo: https://github.com/GarrettLab-UF/LipidMatch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:2fb44d92198951565f6c0ce5c2514871ea28ee60a7befc615f925b1f3867140b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MZmine
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# adduct-aware-lipid-library-querying

## Summary

Queries LipidMatch's in silico precursor-ion libraries (over 250,000 lipid species across 56 lipid types and their adducts) by exact-mass matching each feature's m/z from a peak-picked feature table, at a user-defined m/z tolerance in Da, to generate feature-candidate lipid pairs for downstream MS/MS fragment confirmation.

## When to use

Use when starting lipid annotation from an MZmine/MS-DIAL/XCMS feature table (m/z, RT, intensities) and you need candidate lipid species/adducts matched by precursor exact mass before any MS/MS fragment matching.

## When NOT to use

- Input is still raw vendor MS data — feature detection with MZmine/XCMS/MS-DIAL must precede this MS1 matching step
- Only MS/MS spectral data (.ms2) are available without a peak-picked feature table of precursor m/z values
- Acquisition is on Waters files or lipid classes outside the library coverage (e.g. LipidMatch does not currently support Waters files)

## Inputs

- Feature table CSV from MZmine/MS-DIAL/XCMS containing each feature's m/z, retention time, and intensities
- LipidMatch in silico precursor-ion lipid libraries in .csv format (56 lipid types with polarity-specific adducts)

## Outputs

- Feature-candidate match table (CSV) listing each feature-lipid pair with lipid name, adduct, and library precursor exact mass

## How to apply

Load the feature table CSV produced by peak-picking software (e.g. MZmine) containing each feature's m/z, retention time, and intensities, plus the LipidMatch in silico precursor-ion libraries in .csv format covering the 56 lipid types and their adducts for the given polarity. For each feature, exact-match the feature's m/z against in silico precursor ions using a user-defined m/z tolerance in Da (Step 1 of Fig. 3), restricting the candidate set to the adducts appropriate for the acquisition polarity and excluding dimers, multimers, and in-source fragments. Record each surviving feature-lipid pair with its lipid name, adduct, and library precursor exact mass in a match table saved as CSV. This MS1 step is deliberately permissive because many co-eluting molecules share similar m/z; the candidate list is narrowed later by MS/MS fragment matching within RT and m/z tolerances, so the MS1 library query should over-generate rather than under-generate candidates.

## Related tools

- **LipidMatch** (R-based tool whose in silico precursor-ion libraries and MS1 exact mass matching (Step 1, Fig. 3) produce the feature-candidate matches) — https://github.com/GarrettLab-UF/LipidMatch
- **MZmine** (Peak-picking software generating the feature table (m/z, RT, intensities) that is queried against the lipid libraries)
- **R** (Language in which LipidMatch is implemented (R 3.3.3, gWidgets/tcltk interface) and run to execute the library query) — https://github.com/GarrettLab-UF/LipidMatch

## Evaluation signals

- Every row of the output match table carries lipid name, adduct, and library precursor exact mass alongside the feature's m/z
- Each matched |feature m/z − library precursor exact mass| falls within the user-defined m/z tolerance in Da
- Candidates reflect only adducts valid for the acquisition polarity (positive vs negative mode libraries)
- Dimers, multimers, and in-source fragment ions are absent from the candidate precursor list
- Downstream MS/MS fragment matching should resolve multiple candidates per feature; unresolved ambiguity at this stage is expected rather than an error

## Limitations

- MS1 exact-mass matching alone cannot resolve features containing multiple co-eluting molecules with similar m/z; confirmation requires the later MS/MS fragment steps
- Library coverage defines the candidate space — LipidMatch (2017) covered 56 lipid types (MS-DIAL 34, Greazy 24); custom libraries must be built in Excel for lipids outside coverage
- Software tested only on Windows 7 through 10 and pinned to R version 3.3.3; MS/MS-based confirmation depends on external MSConvert for .ms2 files
- The README reports expanded libraries (over 500,000 lipid species across over 60 lipid types) versus the 2017 paper's over 250,000 across 56 types, so counts differ by distribution version

## Evidence

- [methods] MS1 exact mass matching is user-tolerance based: "LipidMatch performs exact mass matching at the MS1 level between in silico precursor ions and each features m/z using a user defined m/z tolerance (Da) (Step 1; Fig. 3)"
- [methods] Library scope of 56 lipid types: "With 56 lipid types, LipidMatch in silico libraries cover the greatest number of lipid types of any open source software to date, with MS-DIAL containing 34 lipid types, and Greazy containing 24"
- [abstract] Library size stated in abstract: "LipidMatch currently has over 250,000 lipid species spanning 56 lipid types contained in in silico fragmentation libraries."
- [readme] Libraries distributed as CSV: "The link at the bottom of this page contains a manual, lipid libraries in .csv format, a batch file for lipidomics with MZmine processing, and LipidMatch software/scripts."
- [intro] Features may contain co-eluting species: "features (peaks defined by a mass to charge ratio (m/z) and retention time) often contain multiple co-eluting molecules with similar m/z values"
