---
name: lipid-identification-level-assignment
description: Use when downstream interpretation needs to distinguish how structurally
  resolved a lipid annotation is — e.g. separating class-level IDs from fatty-acyl-constituent
  IDs after fragment matching.
when_to_use_negative:
- The input is a raw MS/MS file or feature table rather than the binary fragment matrix
  from Step 4 — the level cannot be assigned without matched-fragment evidence
- The lipid class has no user-defined required-fragment rule, since identification
  then cannot be justified by fragment observation
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: LipidMatch
  role: Performs exact mass matching, fragment matching, and the Step 5/6 rule-based
    identification and level assignment; written in R and invoked through its gWidgets/tcltk
    dialogue boxes
  repo: http://secim.ufl.edu/secim-tools/
- name: MSConvert
  role: 'Upstream: peak picking of MS/MS data and conversion to .ms2 files consumed
    by LipidMatch'
- name: MZmine
  role: 'Upstream: one of the supported peak-picking tools that produce the feature
    tables fed to LipidMatch'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:17c6b98022839266615183d309db7d0d6cd246c03a8317a39cd85592f891393a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidMatch
  - MSConvert
  - MZmine
  - R
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# lipid-identification-level-assignment

## Summary

Assigns each LipidMatch lipid identification an identification level (1–4) reflecting the structural resolution actually conferred by the observed fragments, e.g. PC(16:0_18:1) at level 1 versus PC(32:1) at level 3.

## When to use

Use when downstream interpretation needs to distinguish how structurally resolved a lipid annotation is — e.g. separating class-level IDs from fatty-acyl-constituent IDs after fragment matching.

## When NOT to use

- The input is a raw MS/MS file or feature table rather than the binary fragment matrix from Step 4 — the level cannot be assigned without matched-fragment evidence
- The lipid class has no user-defined required-fragment rule, since identification then cannot be justified by fragment observation

## Inputs

- Binary fragment matrix from LipidMatch Step 4 (fragments scored 0/1 by intensity and scan thresholds)
- In silico lipid library for the queried lipid types
- Excel fragmentation-rules sheet with user-selected required fragments per lipid type

## Outputs

- Output CSV of identified lipids, each row carrying an identification level in 1–4 consistent with observed fragments

## How to apply

After loading the binary fragment matrix (Step 4 output) and the in silico lipid library for the queried lipid types, evaluate the user-selected required-fragment criteria for the lipid class from the Excel fragmentation-rules sheet. Require all necessary fragments to be observed — e.g. both fatty acyl anions for formate-adduct PCs, or the m/z 184.0733 head group for protonated PCs — before calling the lipid identified. Then assign the identification level: 1 for class plus fatty acyl constituents, 2 for data-independent analysis, 3 for class-only, and 4 for precursor m/z without confirmatory fragments. Write each identified lipid with its level to the output CSV.

## Related tools

- **LipidMatch** (Performs exact mass matching, fragment matching, and the Step 5/6 rule-based identification and level assignment; written in R and invoked through its gWidgets/tcltk dialogue boxes) — http://secim.ufl.edu/secim-tools/
- **MSConvert** (Upstream: peak picking of MS/MS data and conversion to .ms2 files consumed by LipidMatch)
- **MZmine** (Upstream: one of the supported peak-picking tools that produce the feature tables fed to LipidMatch)

## Evaluation signals

- Every output identification satisfies the required-fragment rule for its lipid type
- Each row carries a level value in 1–4 consistent with the fragments marked observed in the fragment matrix
- Lipids identified only by precursor m/z without confirmatory fragments are labeled level 4
- Lipids identified by class and fatty acyl constituents (e.g. both fatty acyl anions for formate-adduct PCs) are labeled level 1
- Output CSV rows align with the summed-fragment-intensity ranking produced in Step 7

## Limitations

- Software only tested on Windows 7 through 10; no cross-platform support stated
- Pinned to R version 3.3.3
- Users must develop their own fragmentation libraries in Excel for full customization
- Depends on external MSConvert or equivalent for generating .ms2 files
- Waters files are not currently supported per the README

## Evidence

- [methods] Lipids are identified if they contain the necessary observed fragments.: "Lipids are identified if they contain the necessary observed fragments."
- [methods] they are assigned a number based on whether they are identified by class and fatty acyl constituents (1), by data-independent analysis (2), only by class (3), or only by precursor m/z without: "they are assigned a number based on whether they are identified by class and fatty acyl constituents (1), by data-independent analysis (2), only by class (3), or only by precursor m/z without"
- [methods] Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria or was not found within the m/z tolerance: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria or was not found within the m/z tolerance"
- [abstract] For each lipid type, the user can select which fragments must be observed for identification.: "For each lipid type, the user can select which fragments must be observed for identification."
- [abstract] LipidMatch currently has over 250,000 lipid species spanning 56 lipid types contained in in silico fragmentation libraries.: "LipidMatch currently has over 250,000 lipid species spanning 56 lipid types contained in in silico fragmentation libraries."
- [readme] LipidMatch allows for facile integration of user generated libraries for unique applications.: "LipidMatch allows for facile integration of user generated libraries for unique applications."
