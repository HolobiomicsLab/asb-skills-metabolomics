---
name: lipid-precursor-exact-mass-matching
description: Use when you have a peak-picked feature table (e.g., MZmine CSV with
  m/z, RT, intensities) and in silico lipid precursor libraries, and need candidate
  lipid/adduct assignments per feature before MS/MS fragment confirmation.
when_to_use_negative:
- The goal is MS/MS fragment-level confirmation or identification-level assignment
  (LipidMatch Steps 2-7) — this skill only produces MS1-level candidate matches
- Input is raw vendor MS files rather than a peak-picked feature table; run MZmine/MS-DIAL/XCMS
  feature detection first
- Waters instrument files are involved — the README states LipidMatch does not currently
  support Waters files
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: LipidMatch
  role: R-based tool performing the exact mass matching of feature m/z against in
    silico precursor ions; libraries ship as .csv files
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MZmine
  role: Peak-picking software producing the feature table (m/z, RT, intensities) that
    is matched against the libraries
- name: R
  role: Language in which LipidMatch was written; the matching runs as R scripts behind
    the gWidgets/tcltk dialogue-box UI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:bf900760acfe03982c7b6db0b2dbc0d3b9dfc497380f70857cadd00fcea7e7f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MZmine
  - R
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lipid-precursor-exact-mass-matching

## Summary

Links each peak-picked feature (m/z, RT, intensities) to candidate lipid species by exact mass matching the feature's m/z against in silico precursor ions from LipidMatch's fragmentation libraries at the MS1 level (Step 1 of the LipidMatch workflow), using a user-defined m/z tolerance in Da.

## When to use

Use when you have a peak-picked feature table (e.g., MZmine CSV with m/z, RT, intensities) and in silico lipid precursor libraries, and need candidate lipid/adduct assignments per feature before MS/MS fragment confirmation.

## When NOT to use

- The goal is MS/MS fragment-level confirmation or identification-level assignment (LipidMatch Steps 2-7) — this skill only produces MS1-level candidate matches
- Input is raw vendor MS files rather than a peak-picked feature table; run MZmine/MS-DIAL/XCMS feature detection first
- Waters instrument files are involved — the README states LipidMatch does not currently support Waters files

## Inputs

- Feature table CSV (e.g., from MZmine) containing each feature's m/z, retention time, and intensities
- LipidMatch in silico precursor-ion libraries in .csv format (56 lipid types, adducts, per polarity)

## Outputs

- Feature-candidate match table (CSV) with feature ID, lipid name, adduct, and library precursor exact mass per match

## How to apply

Load the feature table CSV (e.g., exported from MZmine) and the LipidMatch in silico precursor-ion libraries in .csv format, which cover 56 lipid types and their adducts (including oxidized lipids, bile acids, sphingosines, and ammoniated cardiolipins) for the chosen polarity. For each feature, perform exact mass matching at the MS1 level between in silico precursor ions and the feature's m/z using the user-defined m/z tolerance in Da, excluding dimers, multimers, and in-source fragments. Record each feature-lipid pair with the lipid name, adduct, and library precursor exact mass, then save the match table as a CSV. This Step 1 output feeds downstream MS/MS fragment-matching steps (Steps 2-7), so keep the tolerance tight enough to limit candidates but loose enough to retain true precursors.

## Related tools

- **LipidMatch** (R-based tool performing the exact mass matching of feature m/z against in silico precursor ions; libraries ship as .csv files) — https://github.com/GarrettLab-UF/LipidMatch
- **MZmine** (Peak-picking software producing the feature table (m/z, RT, intensities) that is matched against the libraries)
- **R** (Language in which LipidMatch was written; the matching runs as R scripts behind the gWidgets/tcltk dialogue-box UI)

## Evaluation signals

- Every reported feature-candidate pair must have |feature m/z - library precursor exact mass| within the user-defined Da tolerance
- No dimers, multimers, or in-source fragments appear as accepted candidates in the match table
- Each match row contains lipid name, adduct, and library precursor exact mass, not just a lipid label
- Output is a CSV feature-candidate match table usable as input to the MS/MS fragment-matching steps (Step 2 onward)
- Matches are polarity-consistent: positive-mode features only match positive-mode adduct libraries (and vice versa)

## Limitations

- Matching is MS1-only: candidates are not confirmed; confirmation requires MS/MS fragment matching in Steps 2-7
- Too-loose Da tolerances inflate the candidate list per feature; too-tight tolerances miss true precursors — tolerance is user-defined and must be justified
- Only covers lipids in the shipped libraries; users must develop their own fragmentation libraries in Excel for other lipid types
- LipidMatch has been tested on Windows 7 through 10 and pinned to R 3.3.3; no cross-platform support stated

## Evidence

- [methods] Exact mass matching MS1: "LipidMatch performs exact mass matching at the MS1 level between in silico precursor ions and each features m/z using a user defined m/z tolerance (Da) (Step 1; Fig. 3)"
- [abstract] Library scope: "LipidMatch currently has over 250,000 lipid species spanning 56 lipid types contained in in silico fragmentation libraries."
- [readme] Peak-picking integration: "LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer), and combine results from other lipidomics software."
- [methods] Exclusion rules: [evidence span withheld — anchor status: not_found]
