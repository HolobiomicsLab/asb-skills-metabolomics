---
name: chemical-formula-constraint-filtering
description: Use when generating or filtering de novo formula candidates from MS1
  precursor masses and you need to know which element-count restrictions are acceptable,
  or when auditing what fraction of a reference formula set survives candidate-space
  pruning.
when_to_use_negative:
- When scoring or ranking formula candidates against MS/MS spectra — that is MIST-CF's
  FastFilter/neural scoring step, not element-space pruning
- When candidate formulae already come from a database search (e.g. NIST, GNPS, HMDB,
  PubChem queries) rather than de novo mass decomposition
- For negative-mode or adduct-switching analyses — MIST-CF models were trained only
  for positive-mode data and do not consider adduct switching
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Provides the 'SIRIUS decomp' dynamic-programming module whose mass-decomposition
    element constraints (max 5 S, max 3 P, <=1 each halogen) define the filtered candidate
    space
  repo: https://github.com/boecker-lab/sirius
- name: MIST-CF
  role: Downstream consumer of the constrained candidate list; relies on SIRIUS's
    algorithmic decomposition of exact masses into formula candidates
  repo: github.com/samgoldman97/mist-cf
- name: Python
  role: Used to parse formulae into element counts (regex or molmass/Pyteomics) and
    compute the recoverable fraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:2981e287e19ad911826ad81490949790814d9f98550278a18cc9fac9def3c014
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - MIST-CF
  - Python
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# chemical-formula-constraint-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Applies the SIRIUS mass-decomposition element constraints used by 'SIRIUS decomp' (max 5 S atoms, max 3 P atoms, at most one of each halogen F/Cl/Br/I) to restrict de novo chemical formula candidate generation, and audits how much of a reference formula set (e.g. NPLIB1) remains recoverable under those limits.

## When to use

Use when generating or filtering de novo formula candidates from MS1 precursor masses and you need to know which element-count restrictions are acceptable, or when auditing what fraction of a reference formula set survives candidate-space pruning.

## When NOT to use

- When scoring or ranking formula candidates against MS/MS spectra — that is MIST-CF's FastFilter/neural scoring step, not element-space pruning
- When candidate formulae already come from a database search (e.g. NIST, GNPS, HMDB, PubChem queries) rather than de novo mass decomposition
- For negative-mode or adduct-switching analyses — MIST-CF models were trained only for positive-mode data and do not consider adduct switching

## Inputs

- Reference molecular formulae (e.g. NPLIB1/canopus_train formulae from Zenodo record 8151490, canopus_train.zip)
- Element constraint specification: max 5 S, max 3 P, at most one of each halogen (F, Cl, Br, I)

## Outputs

- recoverable_formulae.csv containing the recoverable fraction, counts, and the list of excluded species with their formulae and element counts
- Per-formula classification (recoverable vs. excluded) under the element constraints

## How to apply

Parse each reference molecular formula (e.g. from the NPLIB1/canopus_train dataset downloaded from Zenodo record 8151490) into element counts with a chemistry formula parser (molmass, Pyteomics, or regex-based parsing in Python). Apply the element constraints that MIST-CF inherits from SIRIUS's fast dynamic-programming decomposition: max 5 S, max 3 P, and at most one of each of F, Cl, Br, I; classify each formula as recoverable or excluded. Compute the fraction satisfying the constraints and enumerate the excluded species (e.g. iodixanol) with their element counts. Rationale: MIST-CF reuses 'SIRIUS decomp' for mass decomposition because of its fast implementation, so any element-prior pruning built into that step propagates directly into the MIST-CF candidate list. Validate by confirming the computed recoverable fraction matches the reported ~96% for NPLIB1 (tolerance ±1%) and that the excluded-species list is exported.

## Related tools

- **SIRIUS** (Provides the 'SIRIUS decomp' dynamic-programming module whose mass-decomposition element constraints (max 5 S, max 3 P, <=1 each halogen) define the filtered candidate space) — https://github.com/boecker-lab/sirius
- **MIST-CF** (Downstream consumer of the constrained candidate list; relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates) — github.com/samgoldman97/mist-cf
- **Python** (Used to parse formulae into element counts (regex or molmass/Pyteomics) and compute the recoverable fraction)

## Examples

```
wget https://zenodo.org/record/8151490/files/canopus_train.zip && unzip canopus_train.zip
```

## Evaluation signals

- Computed recoverable fraction of NPLIB1 formulae should be ~96%, consistent with the reported value within ±1%
- Excluded-species list should contain compounds with >5 S, >3 P, or >1 of a single halogen (e.g. iodixanol) with correct element counts
- recoverable_formulae.csv should contain both the recoverable fraction and the excluded-species list
- Every classified formula should parse to element counts consistent with its chemical formula string (parser round-trip check)

## Limitations

- The constraints exclude ~4% of NPLIB1 reference formulae; molecules violating them (e.g. polyhalogenated or sulfur-rich species like iodixanol) can never appear in the candidate list
- MIST-CF still relies on SIRIUS's algorithmic decomposition of exact masses, so candidate space is bounded by SIRIUS's built-in element priors
- MIST-CF models were trained only for positive-mode data and consider multiple adduct types beyond [M+H]+ but still only positive mode

## Evidence
