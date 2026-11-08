---
name: pfas-fragment-library-matching
description: Use when LC-HRMS/MS precursor ions lack exact mass matches to catalogued
  PFAS but their MS/MS may contain diagnostic fluorine-containing fragments (e.g.,
  [SF5]−, [SO3F]−), such as AFFF samples with suspected unknown PFAS.
when_to_use_negative:
- Precursor ions already have exact mass matches to catalogued PFAS — the exact mass
  matching workflow step (before fragment screening) handles those directly
- Input is raw vendor data without the upstream feature detection and blank-filtering
  steps that produce the required feature table
- Analysis is lipidomics rather than PFAS — the Core-Match code defaults to PFAS only
  when the Lipid toggle is FALSE
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FluoroMatch Modular
  role: Hosts the fragment-screening module that reverse-screens precursor MS/MS against
    the fluorine-containing fragment library and writes fragment_assignments.csv
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch 2.0
  role: Overall automated non-targeted PFAS annotation workflow into which the fragment
    library and screening step integrate
  repo: github.com/InnovativeOmics/Core-Match
- name: RCDK
  role: R package whose 'match' function pairs substructures and their associated
    fragments to molecule SMILES when building the SMILES2MSMS.r predicted-fragment
    library
- name: MZedDB
  role: Provides ionisation rules used to restrict the EPA PFAS structure list to
    SMILES capable of forming the [M−H]− precursor ion before library reconstruction
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:e566b7a7a5715337835110e821c76c894f98246bdacf2daab97aafe0a2c7b0fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  - MZedDB
  - RCDK
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pfas-fragment-library-matching

## Summary

Reverse fragment screening in FluoroMatch 2.0/FluoroMatch Modular: search precursor MS/MS against a predicted library of 777 fluorine-containing PFAS fragments (built from the EPA CompTox PFAS structure list via SMILES2MSMS.r) to assign PFAS substructures and flag potential PFAS even when precursor exact masses have no match.

## When to use

Use when LC-HRMS/MS precursor ions lack exact mass matches to catalogued PFAS but their MS/MS may contain diagnostic fluorine-containing fragments (e.g., [SF5]−, [SO3F]−), such as AFFF samples with suspected unknown PFAS.

## When NOT to use

- Precursor ions already have exact mass matches to catalogued PFAS — the exact mass matching workflow step (before fragment screening) handles those directly
- Input is raw vendor data without the upstream feature detection and blank-filtering steps that produce the required feature table
- Analysis is lipidomics rather than PFAS — the Core-Match code defaults to PFAS only when the Lipid toggle is FALSE

## Inputs

- Blank-filtered feature table (CSV) produced by the upstream blank-filtering step
- Raw/candidate precursor MS/MS spectra from LC-HRMS/MS data (e.g., AFFF samples)
- SMILES2MS2_PFAS_fragment_library.csv: predicted fragment–substructure library (777 fluorine-containing PFAS fragments) built from the EPA CompTox Chemicals Dashboard PFAS structure list

## Outputs

- fragment_assignments.csv — per-feature table of matched fluorine-containing fragments and assigned PFAS substructures
- List of flagged potential/unknown PFAS candidates (e.g., features with [SF5]− and [SO3F]− diagnostic fragment assignments)

## How to apply

Load the blank-filtered feature table from upstream feature detection/blank filtering into the FluoroMatch Modular fragment-screening module, and load the reconstructed SMILES2MS2 fragment library (728 substructures / 878 fragments / 777 fluorine-containing, built with SMILES2MSMS.r using the RCDK 'match' function on the EPA CompTox PFAS structure list). Restrict precursors to the PFAS mass-defect window of −0.116 to 0.268 (covering 98% of the EPA list) and the m/z range 117–1189. Reverse-screen each precursor's MS/MS against the 777 fluorine-containing fragments to assign diagnostic fragments and substructures to each feature. Flag features carrying pentafluorosulfide ([SF5]−) and sulfonate ([SO3F]−) diagnostic fragments as unsaturated-PFAS candidates (e.g., the proposed C8 AFFF PFAS class with a pentafluorosulfide and sulfonic acid terminus). Write per-feature assignments to fragment_assignments.csv; exact counts and chemical plausibility of novel classes are left to expert review.

## Related tools

- **FluoroMatch Modular** (Hosts the fragment-screening module that reverse-screens precursor MS/MS against the fluorine-containing fragment library and writes fragment_assignments.csv) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch 2.0** (Overall automated non-targeted PFAS annotation workflow into which the fragment library and screening step integrate) — github.com/InnovativeOmics/Core-Match
- **RCDK** (R package whose 'match' function pairs substructures and their associated fragments to molecule SMILES when building the SMILES2MSMS.r predicted-fragment library)
- **MZedDB** (Provides ionisation rules used to restrict the EPA PFAS structure list to SMILES capable of forming the [M−H]− precursor ion before library reconstruction)

## Examples

```
# In Modular.r (FluoroMatch_Modular): FLOW <- FALSE; csvInput <- FALSE; ManuallyInputVariables <- FALSE; Lipid <- FALSE; TWeen_pos <- FALSE
```

## Evaluation signals

- fragment_assignments.csv exists and parses as a well-formed CSV table
- The C8 candidate row in fragment_assignments.csv contains [SF5]− and [SO3F]− fragment assignments
- The reconstructed library (SMILES2MS2_PFAS_fragment_library.csv) contains known PFAS fragments such as [SF5]− and [C2F5]− together with their substructure SMILES
- Retained precursors fall within the PFAS mass-defect window (−0.116 to 0.268, which captures 98% of the EPA list) and m/z 117–1189
- Species/substructure/fragment counts (728/878/777) and chemical plausibility of novel candidate classes are confirmed by expert review rather than equality assertions

## Limitations

- Fragment/substructure counts are version-dependent and cannot be validated by exact equality assertions — expert review is required
- Chemical plausibility of newly proposed PFAS classes (e.g., the pentafluorosulfide/sulfonic-acid terminus class) is left to expert review; assignments flag candidates, not confirmations
- Substructure matching depends on the RCDK 'match' function and the version of the EPA CompTox PFAS structure list used (e.g., November 2020, >8000 structures)
- In the Core-Match code, the Lipid/TWeen_pos toggles must both be FALSE for PFAS analysis; other settings route to lipidomics behavior

## Evidence
