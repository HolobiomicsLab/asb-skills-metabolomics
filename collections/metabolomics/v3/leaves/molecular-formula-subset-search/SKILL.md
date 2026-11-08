---
name: molecular-formula-subset-search
description: Use when assigning candidate molecular formulas to LC-MS/MS metabolite
  spectra (e.g., mzML from QTOF/ITFT/QFT/Orbitrap datasets) without spectral-library
  matches, or as a fast prescreening step before SIRIUS/Buddy-style identification.
when_to_use_negative:
- 'When accurate structural-level identification is required: FSA does not utilize
  structural information, so accuracy may not be competitive with state-of-the-art
  tools such as Buddy and SIRIUS'
- 'When compounds contain elements beyond C, H, O, N, S, and P, or form diverse adduct
  types: FSA has so far been validated only on CHONSP-containing compounds'
- 'When extensive chemical-rule filtering of formula candidates is essential: FSA
  implements only few chemical rules to rank or filter out unfeasible formulas'
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: 'Core algorithm: computes formula subsets and ranks candidate molecular formulas
    from top-30 fragment peaks per MS/MS spectrum'
- name: ProteoWizard
  role: Free software (msconvert) used to convert vendor raw MS/MS files into open
    formats for downstream FSA analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:7360a15d52248b8a07a8bfc2294830d36bbc24d8746fd02767a57d9b9120739c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# molecular-formula-subset-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs FSA (Formula Subset Analysis), a training-free MS/MS molecular formula ranking algorithm that scores candidate chemical formulas from fragment peaks without requiring MS/MS databases, achieving 85.28% rank-1 accuracy at 0.024 s per spectrum.

## When to use

Use when assigning candidate molecular formulas to LC-MS/MS metabolite spectra (e.g., mzML from QTOF/ITFT/QFT/Orbitrap datasets) without spectral-library matches, or as a fast prescreening step before SIRIUS/Buddy-style identification.

## When NOT to use

- When accurate structural-level identification is required: FSA does not utilize structural information, so accuracy may not be competitive with state-of-the-art tools such as Buddy and SIRIUS
- When compounds contain elements beyond C, H, O, N, S, and P, or form diverse adduct types: FSA has so far been validated only on CHONSP-containing compounds
- When extensive chemical-rule filtering of formula candidates is essential: FSA implements only few chemical rules to rank or filter out unfeasible formulas

## Inputs

- Qualified MS/MS spectra (peak lists) from LC-MS/MS metabolomics datasets, e.g. HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT
- Dataset-specific mass-matching tolerances
- Candidate molecular formulas with monoisotopic molecular weights from PubChem (109,383,767-compound snapshot)

## Outputs

- Per-spectrum ranked list of candidate molecular formulas
- Rank position of the correct/reference formula per spectrum
- Aggregate rank-1 / top-3 / top-5 accuracy percentages (per dataset and overall)

## How to apply

Load qualified MS/MS spectra and select the top-30 most-abundant fragment peaks per spectrum for FSA analysis. Run FSA using dataset-specific mass-matching tolerances against candidate formulas drawn from a public chemical database (PubChem snapshot of 109,383,767 compounds, with formulas and monoisotopic molecular weights); FSA computes molecular formula subspaces without training data or MS/MS databases. Rank candidate formulas per spectrum and record the rank position of the correct formula. Validate outputs by checking rank statistics: across 30,690 spectra / 6,792 metabolites, expect the correct formula at rank #1 for 74.01% of spectra (85.28% of metabolites), in the top-3 for 96.56% of spectra, and in the top-5 for 98.61% of spectra (97.35% of metabolites). Use the ranked formula list as a prescreen or complement to other formula identification tools.

## Related tools

- **FSA** (Core algorithm: computes formula subsets and ranks candidate molecular formulas from top-30 fragment peaks per MS/MS spectrum)
- **ProteoWizard** (Free software (msconvert) used to convert vendor raw MS/MS files into open formats for downstream FSA analysis)

## Examples

```
git clone https://github.com/kslynn128171/FSA && python run_FSA.py --spectra spectra.mzML --top-fragments 30 --tolerance dataset_specific
```

## Evaluation signals

- Overall rank-1 accuracy of the correct formula: ~74.01% of spectra (~85.28% of metabolites) across the 30,690-spectra benchmark
- Top-3 recall: ~96.56% of spectra (~95.27% of metabolites); top-5: ~98.61% of spectra (~97.35% of metabolites)
- Average processing time per spectrum ~0.024 s; deviations far above this on comparable hardware suggest a misconfiguration
- Rank percentages recomputed per dataset (e.g. HMDB_QTOF, CASMI_OT) should match the reported Table 2 values
- No training data or MS/MS database lookup should appear in the run log — FSA requires only chemical formulas and monoisotopic weights

## Limitations

- Rank #1 accuracy not yet comparable to state-of-the-art tools such as Buddy and SIRIUS
- Has not utilized structural information; few chemical rules implemented for ranking/filtering unfeasible formulas, and species/adduct-specific rules are not yet incorporated
- Only validated on compounds composed of C, H, O, N, S, and P
- Employed PubChem snapshot (109,383,767 compounds) is outdated relative to current PubChem versions
- Fragment formulas for peaks without a PubChem match must be derived by adding/subtracting known adducts and neutral losses

## Evidence
