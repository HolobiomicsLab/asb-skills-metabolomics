---
name: elemental-composition-parsing
description: Use when you have raw molecular formula strings from a reference dataset
  (e.g. NPLIB1/canopus_train) and must determine which formulae survive element-count
  constraints imposed by candidate-generation tools like SIRIUS decomp or MIST-CF.
when_to_use_negative:
- Formulae are already provided as parsed element-count tables rather than formula
  strings, making re-parsing redundant
- The goal is scoring candidate formulae against an MS/MS spectrum (that is MIST-CF
  or SIRIUS decomp scoring, not coverage analysis of a reference set)
- The target dataset is not a natural-products formulae set comparable to NPLIB1,
  so the 96% recoverability figure would not transfer
tools:
- name: SIRIUS
  role: Its 'SIRIUS decomp' module defines the dynamic-programming mass decomposition
    and the element constraints (max 5 S, max 3 P, ≤1 each halogen) being screened
    against
  repo: https://github.com/boecker-lab/sirius
- name: molmass/Pyteomics
  role: Chemistry formula parsers suggested for decomposing formula strings into element
    counts
- name: MIST-CF
  role: Downstream consumer of SIRIUS decomp candidate generation, motivating the
    recoverability analysis
  repo: github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:6ff5eee83b14b1e6d6c0337cba1828afbef5f811541cdecf1d975f70aff8fd3b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - MIST-CF
  - Python
  - SIRIUS
  - molmass/Pyteomics
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# elemental-composition-parsing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Parses molecular formula strings (e.g. NPLIB1/GNPS reference formulae) into per-element counts and screens them against the SIRIUS 'SIRIUS decomp' mass-decomposition element constraints (max 5 S, max 3 P, at most one of F/Cl/Br/I) to quantify how many reference formulae remain recoverable under formula-candidate generation limits (~96% of NPLIB1).

## When to use

Use when you have raw molecular formula strings from a reference dataset (e.g. NPLIB1/canopus_train) and must determine which formulae survive element-count constraints imposed by candidate-generation tools like SIRIUS decomp or MIST-CF.

## When NOT to use

- Formulae are already provided as parsed element-count tables rather than formula strings, making re-parsing redundant
- The goal is scoring candidate formulae against an MS/MS spectrum (that is MIST-CF or SIRIUS decomp scoring, not coverage analysis of a reference set)
- The target dataset is not a natural-products formulae set comparable to NPLIB1, so the 96% recoverability figure would not transfer

## Inputs

- NPLIB1 (canopus_train) molecular formula strings, downloadable from Zenodo record 8151490
- Element-count constraint set (max 5 S, max 3 P, at most one each of F, Cl, Br, I) as used by SIRIUS decomp

## Outputs

- recoverable_formulae.csv containing the recoverable fraction, per-formula element counts, and the excluded-species list (e.g. iodixanol)
- Fraction of reference formulae satisfying the element constraints (expected ~96% for NPLIB1)

## How to apply

Download the preprocessed NPLIB1 (canopus_train) data from Zenodo record 8151490, load the reference molecular formulae, and parse each string into element counts using a chemistry parser (molmass/Pyteomics or regex-based Python parsing). Apply the SIRIUS mass-decomposition constraints used by 'SIRIUS decomp' — at most 5 S atoms, at most 3 P atoms, and at most one atom each of F, Cl, Br, and I — classifying each formula as recoverable or excluded. Compute the recoverable fraction and enumerate excluded species (e.g. iodixanol) with their element counts, since MIST-CF relies on SIRIUS's decomposition of exact masses into formula candidates, so any formula violating these constraints can never be generated. Write the fraction, counts, and excluded-species list to a CSV (recoverable_formulae.csv).

## Related tools

- **SIRIUS** (Its 'SIRIUS decomp' module defines the dynamic-programming mass decomposition and the element constraints (max 5 S, max 3 P, ≤1 each halogen) being screened against) — https://github.com/boecker-lab/sirius
- **molmass/Pyteomics** (Chemistry formula parsers suggested for decomposing formula strings into element counts)
- **MIST-CF** (Downstream consumer of SIRIUS decomp candidate generation, motivating the recoverability analysis) — github.com/samgoldman97/mist-cf

## Examples

```
wget https://zenodo.org/record/8151490/files/canopus_train.zip; from molmass import Formula; counts = Formula('C19H26I3N3O9').counts()
```

## Evaluation signals

- Computed recoverable fraction matches the reported ~96% of NPLIB1 formulae within ±1%
- Excluded species (e.g. iodixanol) all violate at least one constraint (multiple halogens, >5 S, or >3 P) with explicit element counts
- Round-trip check: re-serializing parsed element counts reproduces the original formula string (atom conservation)
- recoverable_formulae.csv contains both the fraction and the complete excluded-species list

## Limitations

- The SIRIUS decomp constraints exclude ~4% of NPLIB1 reference formulae, notably multi-halogen species such as iodixanol, so these formulae are unrecoverable for MIST-CF/SIRIUS pipelines
- The recoverability estimate is specific to NPLIB1/GNPS-derived natural products and may differ for other compound classes
- MIST-CF itself was trained only on positive-mode data, so any downstream use of recoverable formulae is restricted to positive-mode spectra

## Evidence
