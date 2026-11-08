---
name: formula-ranking-accuracy-evaluation
description: Use when you have qualified MS/MS spectra paired with known molecular
  formulas and need to quantify how often a formula-ranking tool places the true formula
  at rank#1, within top-3/top-5, and how long it takes per spectrum.
when_to_use_negative:
- The goal is structural identification or spectral similarity matching, not formula
  ranking — FSA has not utilized structural information and its rank#1 accuracy is
  not comparable to state-of-the-art identification tools like Buddy and SIRIUS.
- Compounds contain elements beyond C, H, O, N, S, P or rely on diverse adduct types
  — FSA has only been validated on datasets with compounds composed of C, H, O, N,
  S, and P.
- Spectra lack ground-truth formulas, so rank-of-true-formula cannot be computed for
  accuracy metrics.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: Formula-subset algorithm that generates ranked candidate molecular formulas
    per spectrum using dataset-specific tolerances and top-30 abundant fragments
  repo: https://github.com/kslynn128171/FSA
- name: SIRIUS
  role: State-of-the-art comparator; run SIRIUS 5.8.6 with the paper's optimal ppm
    mass tolerance to produce ranked formulas per spectrum for accuracy and runtime
    comparison
- name: Buddy
  role: State-of-the-art comparator; run Buddy 0.3.8 with 10–20 ppm mass tolerance
    to produce ranked formulas per spectrum for accuracy and runtime comparison
- name: ProteoWizard
  role: msconvert used for free conversion of vendor MS/MS data into qualified spectra
    as upstream preparation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:71b5e7ac8d214a4c28d23b2c6c00bf83836cc8ad1d00e55e08cd85917b253edb
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
  - Buddy
  - FSA
  - ProteoWizard
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# formula-ranking-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark molecular-formula ranking accuracy (rank#1, top-3, top-5) and runtime of formula-assignment tools such as FSA, SIRIUS, and Buddy on MS/MS datasets with ground-truth formulas, reproducing table-style accuracy/speed comparisons.

## When to use

Use when you have qualified MS/MS spectra paired with known molecular formulas and need to quantify how often a formula-ranking tool places the true formula at rank#1, within top-3/top-5, and how long it takes per spectrum.

## When NOT to use

- The goal is structural identification or spectral similarity matching, not formula ranking — FSA has not utilized structural information and its rank#1 accuracy is not comparable to state-of-the-art identification tools like Buddy and SIRIUS.
- Compounds contain elements beyond C, H, O, N, S, P or rely on diverse adduct types — FSA has only been validated on datasets with compounds composed of C, H, O, N, S, and P.
- Spectra lack ground-truth formulas, so rank-of-true-formula cannot be computed for accuracy metrics.

## Inputs

- Qualified MS/MS spectra files with associated ground-truth molecular formulas per dataset (HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT)
- Dataset-specific mass-matching tolerances (ppm) as reported for each dataset
- Tool configurations: SIRIUS 5.8.6 optimal ppm tolerance; Buddy 0.3.8 10–20 ppm tolerance
- Peak-selection parameter: top-30 most-abundant fragment peaks per spectrum

## Outputs

- Per-spectrum rank of the correct molecular formula within each tool's candidate list
- Per-dataset rank#1, top-3, and top-5 accuracy percentages (spectra-level and metabolite-level)
- Per-dataset and total wall-clock processing times, enabling speed-ratio calculations (e.g., FSA vs Buddy/SIRIUS)
- Combined accuracy-and-runtime comparison table analogous to the published Tables 2 and 3

## How to apply

Load qualified MS/MS spectra and ground-truth formulas for each test dataset (e.g., HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT; 30,690 spectra from 6,792 metabolites in the reference study). For each spectrum, restrict input to the top-30 most-abundant fragment peaks and run FSA with the dataset-specific mass-matching tolerance to obtain ranked candidate molecular formulas; record the rank of the correct formula per spectrum. Run comparators on the same data and hardware — SIRIUS 5.8.6 at its optimal ppm tolerance and Buddy 0.3.8 at 10–20 ppm — and record per-spectrum and total wall-clock time. Compute the percentage of spectra (and metabolites) where the correct formula is ranked #1, within top-3, and within top-5, per dataset, then assemble an accuracy-vs-runtime comparison table. Validate against reference values: FSA 74.01% rank#1 for spectra (85.28% for metabolites), 96.56% top-3, 98.61% top-5, with ~148x speed advantage over Buddy and ~134x over SIRIUS (12.5 min vs 1848/1678 min) and ~0.024 s average processing time per spectrum.

## Related tools

- **FSA** (Formula-subset algorithm that generates ranked candidate molecular formulas per spectrum using dataset-specific tolerances and top-30 abundant fragments) — https://github.com/kslynn128171/FSA
- **SIRIUS** (State-of-the-art comparator; run SIRIUS 5.8.6 with the paper's optimal ppm mass tolerance to produce ranked formulas per spectrum for accuracy and runtime comparison)
- **Buddy** (State-of-the-art comparator; run Buddy 0.3.8 with 10–20 ppm mass tolerance to produce ranked formulas per spectrum for accuracy and runtime comparison)
- **ProteoWizard** (msconvert used for free conversion of vendor MS/MS data into qualified spectra as upstream preparation)

## Evaluation signals

- Aggregate rank#1 accuracy over all spectra should reproduce ~74.01% (metabolite-level 85.28%); top-3 ~96.56%; top-5 ~98.61%.
- Computed speed ratios should reproduce ~148x vs Buddy and ~134x vs SIRIUS (FSA ~12.5 min total vs 1848 min Buddy / 1678 min SIRIUS on the same hardware).
- Average per-spectrum processing time for FSA should fall near 0.024 s.
- Per-dataset rank#1/top-3/top-5 accuracies must match the published comparison-table granularity for SIRIUS and Buddy.
- Count of spectra processed should equal 30,690 across the eight datasets.

## Limitations

- FSA does not yet utilize structural information, so its accuracy may not be competitive with some state-of-the-art identification tools (rank#1 below Buddy).
- Few chemical rules have been implemented to rank possible formulas and filter out unfeasible ones; species- and adduct-specific rules are missing, which can cause ambiguity.
- FSA was validated with an outdated PubChem snapshot of 109,383,767 compounds versus 112,419,402 as of 2022/11/27, and only on compounds composed of C, H, O, N, S, and P.
- Dataset-specific tolerances and the top-30 fragment cutoff materially affect results; varying them from the published settings breaks comparability.

## Evidence
