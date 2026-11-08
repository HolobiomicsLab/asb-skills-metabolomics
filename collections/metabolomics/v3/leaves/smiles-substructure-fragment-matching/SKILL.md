---
name: smiles-substructure-fragment-matching
description: Use when you have a catalogued SMILES structure list (e.g., the EPA CompTox
  PFAS list of 8000+ structures) and need predicted fragment ions, including F-containing
  fragments, to drive fragment screening without experimental MS/MS spectra.
when_to_use_negative:
- Experimental MS/MS spectra already exist for the suspect list — the original FluoroMatch
  suspect screening approach targeting compounds with available MS/MS spectra is the
  right path instead
- Input is a feature table or raw LC-HRMS/MS data rather than a SMILES structure list
  — feature detection and blank filtering come first, and this skill operates purely
  on chemical structures
- Analytes ionize in positive mode or outside the MZedDB [M−H]− rules, since the library
  is built by restricting to SMILES forming the [M−H]− precursor ion
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RCDK
  role: Provides the 'match' function used to match substructures and their associated
    fragments to molecule SMILES during library construction
- name: MZedDB
  role: Supplies ionisation rules used to restrict the structure set to SMILES capable
    of forming the [M−H]− precursor ion
- name: FluoroMatch 2.0
  role: Consumes the generated SMILES2MS2 predicted-fragment library for fragment
    screening and predicted MS/MS from SMILES structures in non-targeted PFAS annotation
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular (FLOW <- FALSE) execution environment into which the SMILES2MSMS fragment-prediction
    library output integrates
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:38383e9c99d472d7d1981b892d56e60e777bdf92d278b7e9353f610261ed9d3b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  - MZedDB
  - RCDK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# smiles-substructure-fragment-matching

## Summary

Generate a predicted MS/MS fragment library for PFAS by decomposing a list of SMILES structures into substructures, matching fragments to molecule SMILES with the RCDK 'match' function, and emitting a fragment–substructure table (SMILES2MS2_PFAS_fragment_library.csv) that plugs into FluoroMatch 2.0's predicted-MS/MS-from-SMILES step.

## When to use

Use when you have a catalogued SMILES structure list (e.g., the EPA CompTox PFAS list of 8000+ structures) and need predicted fragment ions, including F-containing fragments, to drive fragment screening without experimental MS/MS spectra.

## When NOT to use

- Experimental MS/MS spectra already exist for the suspect list — the original FluoroMatch suspect screening approach targeting compounds with available MS/MS spectra is the right path instead
- Input is a feature table or raw LC-HRMS/MS data rather than a SMILES structure list — feature detection and blank filtering come first, and this skill operates purely on chemical structures
- Analytes ionize in positive mode or outside the MZedDB [M−H]− rules, since the library is built by restricting to SMILES forming the [M−H]− precursor ion

## Inputs

- SMILES structure list from the EPA CompTox Chemicals Dashboard PFAS structure list (November 2020 version)
- MZedDB ionisation rules for restricting to [M−H]−-capable structures

## Outputs

- SMILES2MS2_PFAS_fragment_library.csv — fragment–substructure library table with exact-mass fragments, substructure SMILES, and fluorine-content flags
- Predicted MS/MS library importable into FluoroMatch 2.0 / FluoroMatch Modular for fragment screening

## How to apply

Start with the EPA CompTox Chemicals Dashboard PFAS structure list (November 2020 version) and restrict it to SMILES capable of forming the [M−H]− precursor ion according to the MZedDB ionisation rules. Decompose each retained SMILES into substructure fragments and map fragment–substructure relationships using the 'match' function from the RCDK package in R — the only input SMILES2MSMS.r requires is this list of SMILES structures. Compute the exact mass of each predicted fragment and flag whether the fragment contains fluorine, then assemble all fragment–substructure records into a single library table written as SMILES2MS2_PFAS_fragment_library.csv. For the reference EPA list this yields 728 substructures, 878 fragments, of which 777 are F-containing. Finally, integrate the output MS/MS library directly into FluoroMatch 2.0, where it supports the fragment screening and predicted MS/MS-from-SMILES workflow steps.

## Related tools

- **RCDK** (Provides the 'match' function used to match substructures and their associated fragments to molecule SMILES during library construction)
- **MZedDB** (Supplies ionisation rules used to restrict the structure set to SMILES capable of forming the [M−H]− precursor ion)
- **FluoroMatch 2.0** (Consumes the generated SMILES2MS2 predicted-fragment library for fragment screening and predicted MS/MS from SMILES structures in non-targeted PFAS annotation) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular (FLOW <- FALSE) execution environment into which the SMILES2MSMS fragment-prediction library output integrates) — github.com/InnovativeOmics/Core-Match

## Evaluation signals

- SMILES2MS2_PFAS_fragment_library.csv exists and parses as a valid CSV table
- The table contains known PFAS fragments such as [SF5]− and [C2F5]− together with their substructure SMILES
- Counts are version-dependent and checked by expert review rather than equality assertions: ~728 substructures, 878 fragments, 777 F-containing fragments for the November 2020 EPA PFAS list
- Every retained input SMILES passes the MZedDB [M−H]− ionisation-rule filter before fragment decomposition
- The assembled library loads directly into FluoroMatch 2.0 and supports its fragment screening step without schema errors

## Limitations

- Exact counts of species, substructures, and fragments are version-dependent and are verified by expert review rather than equality assertions, so numbers change with the EPA PFAS structure-list version
- The library is restricted to SMILES that can form the [M−H]− precursor ion per MZedDB rules; structures outside this ionisation mode are dropped
- Fragments are predicted from SMILES structures only — predicted fragments are not validated against measured MS/MS spectra in this step

## Evidence
