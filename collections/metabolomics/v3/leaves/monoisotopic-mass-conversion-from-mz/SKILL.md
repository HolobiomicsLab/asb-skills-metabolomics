---
name: monoisotopic-mass-conversion-from-mz
description: Use when LC-MS/MS precursor or fragment m/z values must be converted
  to neutral monoisotopic masses before formula enumeration or adduct-aware mass matching
  (as FSA does against PubChem-derived formula lists).
when_to_use_negative:
- The input is already monoisotopic neutral masses or a feature table with formula-level
  annotations, not raw m/z values
- The goal is in silico MS/MS spectrum prediction or spectral matching against databases
  (e.g. CFM-ID 4.0 vs METLIN), not formula-space preprocessing
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: Implements the m/z-to-monoisotopic-mass conversion as the preprocessing step
    before fragment formula searching
  repo: https://github.com/kslynn128171/FSA
- name: ProteoWizard
  role: msconvert can be used to obtain the MS/MS data that this conversion is applied
    to
- name: UC Davis adduct calculator
  role: Reference for correcting common adduct masses when converting m/z to neutral
    monoisotopic mass
  repo: https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:8db213b0978ba0f3e848bfe81b98a7e796cb29db29556356cd39140b91d67a8e
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
  - UC Davis adduct calculator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# monoisotopic-mass-conversion-from-mz

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing step of the FSA (formula subset analysis) metabolite identification workflow in which raw MS/MS m/z values are converted to monoisotopic masses so that precursor and fragment chemical formulas can be searched within a mass matching tolerance.

## When to use

Use when LC-MS/MS precursor or fragment m/z values must be converted to neutral monoisotopic masses before formula enumeration or adduct-aware mass matching (as FSA does against PubChem-derived formula lists).

## When NOT to use

- The input is already monoisotopic neutral masses or a feature table with formula-level annotations, not raw m/z values
- The goal is in silico MS/MS spectrum prediction or spectral matching against databases (e.g. CFM-ID 4.0 vs METLIN), not formula-space preprocessing

## Inputs

- MS/MS spectra with precursor and fragment m/z values (LC-MS/MS metabolomics data, e.g. as processed with ProteoWizard msconvert)
- Optional: expected ion/adduct type for the precursor

## Outputs

- Monoisotopic neutral mass per precursor and per fragment, replacing raw m/z values as input to formula searching
- Converted spectra feeding downstream FSA steps (fragment formula search, MDR enumeration, Eq. (4) ranking)

## How to apply

As the first step of the FSA workflow, load each MS/MS spectrum and convert every fragment m/z value (and the precursor m/z) to its monoisotopic mass, per the FSA Methods description. Rationale: formula searches against PubChem-derived reference formula lists operate on neutral monoisotopic molecular weights, not observed m/z, so this conversion must happen before enumerating candidate formulas within a given mass matching tolerance. When the ion is expected to be an adduct rather than a bare molecule, account for common adduct masses using a reference calculator such as the UC Davis adduct calculator. Handle isotope ambiguity by deriving the monoisotopic peak from the observed m/z so subsequent mass-difference (MDR) computations among fragment formulas are consistent. Downstream outputs of this step feed fragment formula search, mass-difference relationship enumeration, and Eq. (4) precursor formula scoring.

## Related tools

- **FSA** (Implements the m/z-to-monoisotopic-mass conversion as the preprocessing step before fragment formula searching) — https://github.com/kslynn128171/FSA
- **ProteoWizard** (msconvert can be used to obtain the MS/MS data that this conversion is applied to)
- **UC Davis adduct calculator** (Reference for correcting common adduct masses when converting m/z to neutral monoisotopic mass) — https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/

## Evaluation signals

- Every fragment m/z in the spectrum maps to a monoisotopic mass before formula search; no raw m/z values remain in downstream FSA inputs
- The converted precursor mass, matched against the PubChem-derived formula list within the mass tolerance, returns a nonempty candidate formula set
- Mass differences among fragment formulas (used for MDR enumeration) are consistent with neutral monoisotopic-mass arithmetic, not m/z arithmetic
- Running the standalone FSA module on the same inputs reproduces the paper-reported ranking behavior (top-1 accuracy 85.28%, top-5 accuracy 97.35% on the benchmark spectra from the eight test datasets) within reasonable tolerance

## Limitations

- FSA has been validated only on compounds composed of C, H, O, N, S, and P; the conversion-plus-formula-search pipeline is unvalidated for molecules beyond these elements
- Compounds forming various adduct types are not yet handled; species/adduct-specific rules are not incorporated, so a single fixed conversion assumption may be wrong for unusual adducts
- The PubChem reference snapshot used by FSA is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), which bounds what converted masses can be matched against

## Evidence

- [methods] FSA preprocesses the spectrum by converting m/z to monoisotopic masses, then searches possible chemical formulas for the precursor and each fragment under a given mass matching tolerance.: [evidence span withheld — anchor status: not_found]
