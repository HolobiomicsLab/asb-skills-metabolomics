---
name: rcdk-molecular-structure-parsing
description: Use when you need to generate predicted MS/MS fragment libraries from
  a list of SMILES structures — e.g. building substructure-to-fragment mappings for
  non-targeted PFAS annotation when no experimental MS/MS spectra exist for the candidates.
when_to_use_negative:
- Experimental MS/MS spectra already exist for the candidate compounds — use the original
  FluoroMatch suspect-screening approach targeting compounds with available spectra
  instead
- Structures cannot form the [M−H]− precursor ion under MZedDB ionisation rules and
  are outside the target negative-mode PFAS workflow
- The goal is feature-level data processing (feature detection, blank filtering, mass
  defect filtering) rather than fragment-library construction
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: RCDK
  role: R package whose 'match' function performs the substructure-to-molecule SMILES
    matching that generates fragment–substructure relationships
- name: SMILES2MSMS.r
  role: Script that wraps the RCDK matching to build the predicted MS/MS fragment
    library from a SMILES input list
- name: MZedDB
  role: Ionisation-rule reference used to restrict the SMILES set to structures that
    can form the [M−H]− precursor ion
- name: FluoroMatch 2.0
  role: Downstream software into which the generated SMILES2MS2 PFAS fragment library
    integrates for non-targeted PFAS annotation
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:861b21df1990070cb00bf2a33b911d3a293058c3e595f0ba6ff62b0eb7b9d8cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - MZedDB
  - RCDK
  - SMILES2MSMS.r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rcdk-molecular-structure-parsing

## Summary

Decomposes SMILES structures into substructure fragments and maps fragment–substructure relationships using the RCDK package in R, powering the SMILES2MSMS.r predicted-fragment library for PFAS. It turns catalogued structure lists (e.g. the EPA CompTox PFAS list) into exact-mass fragment tables usable for predicted MS/MS screening.

## When to use

Use when you need to generate predicted MS/MS fragment libraries from a list of SMILES structures — e.g. building substructure-to-fragment mappings for non-targeted PFAS annotation when no experimental MS/MS spectra exist for the candidates.

## When NOT to use

- Experimental MS/MS spectra already exist for the candidate compounds — use the original FluoroMatch suspect-screening approach targeting compounds with available spectra instead
- Structures cannot form the [M−H]− precursor ion under MZedDB ionisation rules and are outside the target negative-mode PFAS workflow
- The goal is feature-level data processing (feature detection, blank filtering, mass defect filtering) rather than fragment-library construction

## Inputs

- List of SMILES structures (e.g. EPA CompTox Chemicals Dashboard PFAS structure list, November 2020 version)
- MZedDB ionisation rules for restricting to structures capable of forming the [M−H]− precursor ion

## Outputs

- SMILES2MS2_PFAS_fragment_library.csv — table of fragment–substructure records with exact masses and fluorine-content flags
- Predicted MS/MS library integrated into FluoroMatch 2.0

## How to apply

Start from a SMILES structure list such as the EPA CompTox Chemicals Dashboard PFAS structure list (November 2020 version, >8000 structures), and restrict it to SMILES capable of forming the [M−H]− precursor ion according to MZedDB ionisation rules. Within SMILES2MSMS.r, use the 'match' function from the RCDK package in R to match substructures and their associated fragments to each molecule SMILES, decomposing each retained SMILES into substructure fragments and mapping the fragment–substructure relationships. For every predicted fragment, compute the exact mass and flag whether the fragment contains fluorine (the November 2020 EPA list yields 728 substructures, 878 fragments, 777 F-containing). Assemble the fragment–substructure records into a single library table written as SMILES2MS2_PFAS_fragment_library.csv, which integrates directly into FluoroMatch 2.0 for predicted MS/MS screening from SMILES structures.

## Related tools

- **RCDK** (R package whose 'match' function performs the substructure-to-molecule SMILES matching that generates fragment–substructure relationships)
- **SMILES2MSMS.r** (Script that wraps the RCDK matching to build the predicted MS/MS fragment library from a SMILES input list)
- **MZedDB** (Ionisation-rule reference used to restrict the SMILES set to structures that can form the [M−H]− precursor ion)
- **FluoroMatch 2.0** (Downstream software into which the generated SMILES2MS2 PFAS fragment library integrates for non-targeted PFAS annotation) — github.com/InnovativeOmics/Core-Match

## Evaluation signals

- The output file SMILES2MS2_PFAS_fragment_library.csv exists and parses as a valid CSV table
- Known PFAS fragments such as [SF5]− and [C2F5]− are present together with their substructure SMILES
- Every fragment record carries an exact mass and a fluorine-content flag; the F-containing fraction is high (777 of 878 fragments for the Nov 2020 EPA list)
- Structures unable to form [M−H]− under MZedDB rules are excluded from the library
- Exact counts of species, substructures, and fragments are version-dependent and therefore verified by expert review rather than equality assertions

## Limitations

- The exact counts of species, substructures, and fragments are version-dependent (e.g. 728 substructures / 878 fragments for the November 2020 EPA PFAS list) and cannot be asserted as fixed equality checks
- The library only covers structures that can form the [M−H]− precursor ion according to MZedDB ionisation rules
- Integration instructions live in distributed manuals (FluoroMatch_Flow_Manual.docx, FluoroMatch_Modular/FluoroMatch_Manual.docx); developers must replace Modular.r in LipidMatch-4.2 directories when editing Core-Match code

## Evidence

- [other] SMILES2MSMS.r requires only a list of SMILES structures; it matches substructures and their associated fragments to molecule SMILES via the 'match' function from the RCDK package in R: [evidence span withheld — anchor status: not_found]
