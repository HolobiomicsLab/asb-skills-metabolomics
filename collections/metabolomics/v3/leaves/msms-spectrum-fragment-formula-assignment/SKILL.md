---
name: msms-spectrum-fragment-formula-assignment
description: Use when assigning/ranking molecular formulas for deposited LC-MS/MS
  spectra (e.g., HMDB or NPL datasets) with dataset-specific mass-matching tolerances
  and the top-30 most-abundant fragment peaks, without relying on MS/MS spectral databases.
when_to_use_negative:
- Spectra containing elements beyond C, H, O, N, S, and P — FSA has only been validated
  on compounds composed of these elements
- Analyses requiring structural identification or state-of-the-art rank#1 accuracy
  comparable to SIRUS or Buddy, since FSA does not use structural information
- Compounds analyzed as various adduct types, for which species/adduct-specific chemical
  rules are not yet incorporated
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: Formula-subset algorithm that ranks candidate molecular formulas from fragment
    peaks; invoked via the kslynn128171/FSA repository
  repo: https://github.com/kslynn128171/FSA
- name: ProteoWizard
  role: msconvert used for converting raw vendor MS/MS data into open formats prior
    to FSA analysis
- name: PubChem
  role: Source of chemical formulas and monoisotopic molecular weights for candidate
    formula generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:277a0f39d4a4a104cecfe092fe94a581f97f66fa234dcb80bdaf109cc3c3cea7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - ProteoWizard
  - PubChem
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# msms-spectrum-fragment-formula-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank candidate molecular formulas for LC-MS/MS metabolite spectra using the FSA formula-subset algorithm, which matches top-abundant fragment peaks against database-derived chemical formulas and monoisotopic molecular weights rather than MS/MS training data.

## When to use

Use when assigning/ranking molecular formulas for deposited LC-MS/MS spectra (e.g., HMDB or NPL datasets) with dataset-specific mass-matching tolerances and the top-30 most-abundant fragment peaks, without relying on MS/MS spectral databases.

## When NOT to use

- Spectra containing elements beyond C, H, O, N, S, and P — FSA has only been validated on compounds composed of these elements
- Analyses requiring structural identification or state-of-the-art rank#1 accuracy comparable to SIRUS or Buddy, since FSA does not use structural information
- Compounds analyzed as various adduct types, for which species/adduct-specific chemical rules are not yet incorporated

## Inputs

- Qualified MS/MS spectra from the eight deposited test datasets (HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT)
- Ground-truth chemical formulas for each spectrum/metabolite
- Dataset-specific mass-matching tolerances
- Monoisotopic molecular weights and chemical formulas from databases such as PubChem (109,383,767-compound snapshot used in the study) and ChemSpider

## Outputs

- Ranked candidate molecular formula list per spectrum
- Rank of the correct formula per spectrum
- Per-dataset and overall percentages of spectra/metabolites with correct formula at rank#1, top-3, and top-5 (e.g., 74.01%/85.28% rank#1, 96.56% top-3, 98.61%/97.35% top-5)

## How to apply

Load qualified MS/MS spectra and ground-truth formulas, then select the top-30 most-abundant fragment peaks per spectrum as FSA input. Run FSA using dataset-specific mass-matching tolerances over formulas/monoisotopic weights drawn from chemical databases such as PubChem and ChemSpider, restricted in this study to CHONSP-element compounds. FSA outputs a ranked list of candidate molecular formulas per spectrum in ~0.024 s per spectrum; record the rank of the correct formula. Evaluate by computing the fraction of spectra where the correct formula is rank#1, in the top-3, and in the top-5 (reported benchmarks: 85.28% top-1 and 97.35% top-5 across 30,690 spectra from 6792 metabolites). Compare computed per-dataset percentages against the paper's Table 2 values as a validation check.

## Related tools

- **FSA** (Formula-subset algorithm that ranks candidate molecular formulas from fragment peaks; invoked via the kslynn128171/FSA repository) — https://github.com/kslynn128171/FSA
- **ProteoWizard** (msconvert used for converting raw vendor MS/MS data into open formats prior to FSA analysis)
- **PubChem** (Source of chemical formulas and monoisotopic molecular weights for candidate formula generation)

## Evaluation signals

- Overall rank#1 accuracy should reproduce ~85.28% of metabolites (74.01% of spectra) and top-5 accuracy ~97.35% of metabolites (98.61% of spectra) across the 30,690-spectrum benchmark
- Per-dataset rank percentages should match the reported Table 2 values for each of the eight datasets
- Average processing time per spectrum should be on the order of 0.024 s
- Each spectrum must use exactly the top-30 most-abundant fragment peaks and the dataset-specific tolerance from the paper
- Output must be a ranked formula list per spectrum containing the ground-truth formula as a checkable entry

## Limitations

- Rank#1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS (e.g., SIRIUS took 5 h 41 min for 5518 spectra vs FSA's 0.024 s/spectrum), because FSA does not utilize structural information
- Only validated on CHONSP-element compounds; extension to more complex molecules and multiple adduct types is future work
- Uses an outdated PubChem snapshot (109,383,767 vs 112,419,402 compounds as of 2022/11/27)
- Few chemical rules are implemented to rank/filter unfeasible formulas, leaving ambiguity in certain compound species and adduct cases

## Evidence
