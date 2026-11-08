---
name: spectral-peak-abundance-filtering
description: Use when preparing qualified LC-MS/MS spectra (e.g., from HMDB_QTOF,
  NPL_OT, CASMI_OT datasets) for formula ranking with FSA and a peak-count cutoff
  of 30 is needed.
when_to_use_negative:
- When the goal is full molecular structure elucidation requiring structural information
  — FSA 'has not utilized structural information yet' and its accuracy is not competitive
  with tools like Buddy or SIRIUS
- When compounds contain elements beyond C, H, O, N, S, and P — FSA has only been
  validated on those elements
- When the spectrum is from a mixture where fragment patterns differ from a single
  compound, as this leads to differences between predicted and actual fragment behavior
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FSA
  role: Formula-subset algorithm applied to the top-30 abundance-filtered peaks to
    generate ranked candidate molecular formulas
  repo: https://github.com/kslynn128171/FSA
- name: ProteoWizard
  role: msconvert from ProteoWizard cited in the article as free software for converting
    vendor MS/MS data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:995f03a4e3f615bf941c790905a3eba8c18368d1d7fc98ba3fe2f446bb7b70c4
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectral-peak-abundance-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Select the top-30 most-abundant fragment peaks from each qualified MS/MS spectrum before feeding FSA's formula-subset analysis, the preprocessing step that reproducibly yields the paper's reported ranking accuracies (85.28% rank-#1, 97.35% top-5).

## When to use

Use when preparing qualified LC-MS/MS spectra (e.g., from HMDB_QTOF, NPL_OT, CASMI_OT datasets) for formula ranking with FSA and a peak-count cutoff of 30 is needed.

## When NOT to use

- When the goal is full molecular structure elucidation requiring structural information — FSA 'has not utilized structural information yet' and its accuracy is not competitive with tools like Buddy or SIRIUS
- When compounds contain elements beyond C, H, O, N, S, and P — FSA has only been validated on those elements
- When the spectrum is from a mixture where fragment patterns differ from a single compound, as this leads to differences between predicted and actual fragment behavior

## Inputs

- Qualified MS/MS spectra from LC-MS/MS metabolomics test datasets (with ground-truth molecular formulas per metabolite)
- Fragment peak lists with intensity/abundance values per spectrum

## Outputs

- Per-spectrum subset of the top-30 most-abundant fragment peaks used for formula-subset analysis
- Ranked candidate molecular formulas per spectrum with the rank of the correct formula recorded

## How to apply

For each spectrum in an eight-dataset test set (HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT), rank fragment peaks by abundance and keep only the top-30 for FSA analysis. This abundance-based subset removes noisy, low-signal peaks that would expand the candidate formula space without improving ranking, and it is the exact configuration used with dataset-specific mass-matching tolerances to reproduce Table 2. After filtering, run the FSA algorithm on the retained peaks to generate ranked candidate molecular formulas, then record the rank of the ground-truth formula per spectrum. Validate by computing top-1, top-3, and top-5 hit percentages and comparing against the reported values (74.01%/85.28% top-1 for spectra/metabolites; 96.56%/95.27% top-3; 98.61%/97.35% top-5).

## Related tools

- **FSA** (Formula-subset algorithm applied to the top-30 abundance-filtered peaks to generate ranked candidate molecular formulas) — https://github.com/kslynn128171/FSA
- **ProteoWizard** (msconvert from ProteoWizard cited in the article as free software for converting vendor MS/MS data)

## Evaluation signals

- Per dataset, the rank-#1 accuracy across spectra should approximate 74.01% (85.28% at the metabolite level) when the top-30 filter and dataset-specific tolerances are used
- Top-3 accuracy should approximate 96.56% of spectra and top-5 accuracy 98.61% of spectra (95.27% and 97.35% at the metabolite level)
- Aggregate validation should cover 30,690 spectra from 6792 metabolites across the eight deposited test datasets
- Each retained peak subset must contain no more than 30 fragment peaks ranked by abundance
- FSA average processing time per spectrum should remain near the reported 0.024 s

## Limitations

- Few chemical rules are implemented to rank possible formulas or filter out unfeasible ones; species/adduct-specific rules are not yet incorporated, which can be crucial for certain compound classes and adducts
- The PubChem snapshot embedded in FSA is outdated (109,383,767 compounds vs. 112,419,402 as of 2022/11/27), so newer formulas may be missed
- Rank-#1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS; including more chemical formula databases (ChemSpider, ChEBI, SciFinder) may be needed

## Evidence

- [results] Across 30,690 spectra in the eight test datasets, FSA (using dataset-specific tolerances and top-30 abundant fragments) ranked the correct formula at rank#1 for 74.01% of spectra: [evidence span withheld — anchor status: not_found]
- [methods] Load the qualified MS/MS spectra and ground-truth formulas for each of the eight deposited test datasets: [evidence span withheld — anchor status: not_found]
- [methods] For each spectrum, select the top-30 most-abundant fragment peaks for FSA analysis.: [evidence span withheld — anchor status: not_found]
