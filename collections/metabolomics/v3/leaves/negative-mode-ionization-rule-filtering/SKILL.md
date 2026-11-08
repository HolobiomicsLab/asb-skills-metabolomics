---
name: negative-mode-ionization-rule-filtering
description: Use when converting a catalogued SMILES list (e.g., the EPA CompTox PFAS
  list) into a predicted MS/MS fragment library for negative-mode PFAS annotation
  and you must drop structures incapable of forming [M−H]−.
when_to_use_negative:
- Compounds with measured MS/MS spectra already available — the original FluoroMatch
  suspect-screening workflow targets those, so predicted-fragment generation is redundant
- Positive-mode acquisition campaigns — the MZedDB rule set applied here is specific
  to [M−H]− precursor formation and would wrongly discard structures ionizable only
  in positive mode
- Input is already a feature table or peak list without SMILES structures — ionization
  rules cannot be evaluated without structural records
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MZedDB
  role: Supplies the ionisation rules used to decide whether a SMILES can form the
    [M−H]− precursor ion
- name: RCDK
  role: R package whose 'match' function maps substructures and fragments onto the
    filtered molecule SMILES to build the predicted-fragment library
- name: FluoroMatch 2.0
  role: Consumes the filtered, SMILES-derived MS/MS library for predicted MS/MS screening
    in non-targeted PFAS annotation
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular distribution in which the edited algorithms (Modular.r) are integrated;
    the filtered library feeds its predicted MS/MS workflow
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:db971da078a0744b1076161f29fb4361d33c27db521114d79730ab64e1f029cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  - MZedDB
  - RCDK
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# negative-mode-ionization-rule-filtering

## Summary

Restricts a candidate PFAS structure list to SMILES that can form the [M−H]− precursor ion per MZedDB ionisation rules, so that predicted-MS/MS fragment libraries generated downstream (via RCDK substructure matching) are physically ionizable in negative-mode LC–HRMS experiments used by FluoroMatch 2.0.

## When to use

Use when converting a catalogued SMILES list (e.g., the EPA CompTox PFAS list) into a predicted MS/MS fragment library for negative-mode PFAS annotation and you must drop structures incapable of forming [M−H]−.

## When NOT to use

- Compounds with measured MS/MS spectra already available — the original FluoroMatch suspect-screening workflow targets those, so predicted-fragment generation is redundant
- Positive-mode acquisition campaigns — the MZedDB rule set applied here is specific to [M−H]− precursor formation and would wrongly discard structures ionizable only in positive mode
- Input is already a feature table or peak list without SMILES structures — ionization rules cannot be evaluated without structural records

## Inputs

- EPA CompTox Chemicals Dashboard PFAS structure list (SMILES strings, November 2020 version)
- MZedDB ionisation rule set for negative-mode ([M−H]−) precursor formation

## Outputs

- Filtered SMILES set restricted to [M−H]−-capable structures
- SMILES2MS2_PFAS_fragment_library.csv containing fragment–substructure records with exact masses and fluorine-content flags (728 substructures, 878 fragments, 777 F-containing for the reference build)

## How to apply

Start from the EPA CompTox Chemicals Dashboard PFAS structure list (November 2020 version) and apply the MZedDB ionisation rules to keep only SMILES capable of forming the [M−H]− precursor ion; this is a filter step, not a fragmenting step, so no fragmentation chemistry is needed here. Pass the retained SMILES to the SMILES2MSMS.r workflow, where RCDK's 'match' function pairs each molecule SMILES with substructures and their associated fragments. Compute each predicted fragment's exact mass and flag fluorine-containing fragments (777 of 878 fragments in the reference build). Assemble fragment–substructure records into SMILES2MS2_PFAS_fragment_library.csv for direct import into FluoroMatch 2.0/FluoroMatch Modular. The rationale is that a structure that cannot ionize as [M−H]− would otherwise generate fragment annotations that can never match a real precursor in negative-mode data, inflating false candidate matches.

## Related tools

- **MZedDB** (Supplies the ionisation rules used to decide whether a SMILES can form the [M−H]− precursor ion)
- **RCDK** (R package whose 'match' function maps substructures and fragments onto the filtered molecule SMILES to build the predicted-fragment library)
- **FluoroMatch 2.0** (Consumes the filtered, SMILES-derived MS/MS library for predicted MS/MS screening in non-targeted PFAS annotation) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular distribution in which the edited algorithms (Modular.r) are integrated; the filtered library feeds its predicted MS/MS workflow) — github.com/InnovativeOmics/Core-Match

## Evaluation signals

- Output file SMILES2MS2_PFAS_fragment_library.csv exists and parses as a well-formed CSV table
- Library contains known PFAS fragments such as [SF5]− and [C2F5]− together with their substructure SMILES
- For the November 2020 EPA build, expected counts are 728 substructures, 878 fragments, and 777 F-containing fragments (version-dependent, checked by expert review rather than equality assertions)
- Every retained fragment record carries a computed exact mass and a fluorine flag consistent with its SMILES
- Filtered structures are all compatible with [M−H]− formation so downstream predicted MS/MS integrates into FluoroMatch 2.0 without unmatched-precursor errors

## Limitations

- Exact species/substructure/fragment counts are version-dependent and are validated by expert review rather than equality assertions
- The rule set covers only [M−H]− formation, so positive-mode PFAS candidates are excluded by design
- Coverage is bounded by the catalogued EPA PFAS structure list (over 8000 structures); truly unknown PFAS outside the list cannot be predicted
- Code edits must be manually integrated into the InnovativeOmics distribution directories (e.g., LipidMatch-4.2\FluoroMatch_Modular) — the GitHub Core-Match repo is for developer editing, not the stable release

## Evidence

- [other] MZedDB [M−H]− restriction rule: [evidence span withheld — anchor status: not_found]
