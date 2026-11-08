---
name: fragment-exact-mass-calculation
description: Use when a list of PFAS SMILES structures must be converted into a predicted
  fragment library (exact masses, fluorine flags, substructure mapping) for fragment-based
  non-targeted screening in FluoroMatch 2.0.
when_to_use_negative:
- Target compounds already have experimental MS/MS spectra available — the original
  FluoroMatch suspect-screening approach (matching against available MS/MS spectra)
  is the appropriate path instead
- Input is a processed feature table or spectra rather than SMILES structures — fragment
  prediction requires the chemical structure list as input
- The analysis targets non-PFAS analytes — this fragment library is built specifically
  from the EPA PFAS structure list and defaults to PFAS analysis
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0171
- http://edamontology.org/topic_3318
tools:
- name: FluoroMatch 2.0
  role: 'Downstream consumer: integrates the predicted MS/MS fragment library into
    its fragment screening workflow for non-targeted PFAS annotation'
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular R framework in which the fragment library is generated and plugged
    in for predicted-MS/MS-from-SMILES screening
  repo: github.com/InnovativeOmics/Core-Match
- name: RCDK
  role: Used in R to decompose SMILES into substructures and match substructure fragments
    to molecule SMILES via its 'match' function
- name: MZedDB
  role: Provides ionisation rules used to restrict the structure set to SMILES capable
    of forming the [M−H]− precursor ion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:7b4ed26434833a13bce8e320a8589e63862873166f176e7b71c13d0d1960e36d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  - MZedDB
  - RCDK
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0171
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# fragment-exact-mass-calculation

## Summary

Computes exact masses for substructure-derived PFAS fragments predicted from SMILES structures (via RCDK), flags fluorine-containing fragments, and assembles them into a predicted MS/MS fragment library consumable by FluoroMatch 2.0.

## When to use

Use when a list of PFAS SMILES structures must be converted into a predicted fragment library (exact masses, fluorine flags, substructure mapping) for fragment-based non-targeted screening in FluoroMatch 2.0.

## When NOT to use

- Target compounds already have experimental MS/MS spectra available — the original FluoroMatch suspect-screening approach (matching against available MS/MS spectra) is the appropriate path instead
- Input is a processed feature table or spectra rather than SMILES structures — fragment prediction requires the chemical structure list as input
- The analysis targets non-PFAS analytes — this fragment library is built specifically from the EPA PFAS structure list and defaults to PFAS analysis

## Inputs

- EPA CompTox Chemicals Dashboard PFAS structure list (SMILES structures of catalogued PFAS, November 2020 version)
- MZedDB ionisation rules (for restricting to [M−H]−-capable structures)
- Fragment–substructure association definitions from the SMILES2MSMS.r library

## Outputs

- SMILES2MS2_PFAS_fragment_library.csv — table of predicted fragment–substructure records with computed exact masses and fluorine-containment flags
- Predicted MS/MS library integrable into FluoroMatch 2.0 (covering 728 substructures, 878 fragments, 777 F-containing fragments for this PFAS version)

## How to apply

Load the EPA CompTox Chemicals Dashboard PFAS structure list (November 2020 version, >8000 catalogued PFAS SMILES) and restrict it to structures capable of forming the [M−H]− precursor ion according to MZedDB ionisation rules. For each retained SMILES, decompose the structure into substructures and match substructures and their associated fragments to the molecule SMILES using the 'match' function from the RCDK package in R. Compute the exact mass of each predicted fragment and flag whether the fragment contains fluorine, since F-containing fragments (e.g. [SF5]−, [C2F5]−) carry diagnostic weight in PFAS annotation. Assemble the fragment–substructure records into a single library table (SMILES2MS2_PFAS_fragment_library.csv) so it integrates directly into FluoroMatch 2.0's fragment screening workflow. Exact counts are version-dependent and should be validated by expert review rather than equality assertions.

## Related tools

- **FluoroMatch 2.0** (Downstream consumer: integrates the predicted MS/MS fragment library into its fragment screening workflow for non-targeted PFAS annotation) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular R framework in which the fragment library is generated and plugged in for predicted-MS/MS-from-SMILES screening) — github.com/InnovativeOmics/Core-Match
- **RCDK** (Used in R to decompose SMILES into substructures and match substructure fragments to molecule SMILES via its 'match' function)
- **MZedDB** (Provides ionisation rules used to restrict the structure set to SMILES capable of forming the [M−H]− precursor ion)

## Evaluation signals

- Output file SMILES2MS2_PFAS_fragment_library.csv exists and is parseable as a CSV table
- Known PFAS fragments such as [SF5]− and [C2F5]− appear in the output together with their substructure SMILES
- Every fragment record carries a computed exact mass and a fluorine-containment flag (expected 777 of 878 fragments F-containing for the November 2020 EPA PFAS list)
- All retained precursor structures satisfy the MZedDB ionisation rules for [M−H]− formation
- Fragment/substructure counts (728 substructures, 878 fragments) are consistent with the corresponding EPA PFAS structure-list version, confirmed by expert review rather than equality assertions

## Limitations

- Exact counts of species, substructures, and fragments are version-dependent; they change with the EPA PFAS structure-list version and are validated by expert review rather than fixed equality assertions
- Fragments are predicted from SMILES substructures rather than observed spectra, so predictions must pass downstream fragment screening against real MS/MS data before annotation confidence is assigned
- Only structures capable of forming the [M−H]− precursor ion per MZedDB rules are retained; structures outside those ionisation rules are excluded

## Evidence

- [other] Compute the exact mass of each predicted fragment and flag whether the fragment contains fluorine.: [evidence span withheld — anchor status: not_found]
- [other] matches substructures and their associated fragments to molecule SMILES via the 'match' function from the RCDK package: [evidence span withheld — anchor status: not_found]
