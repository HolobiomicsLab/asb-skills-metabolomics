---
name: element-count-feasibility-verification
description: Use when evaluating whether an element-constrained formula generation
  space (SIRIUS decomp-style limits) excludes a meaningful share of reference formulae
  from a spectral dataset such as NPLIB1 before benchmarking de novo formula predictors.
when_to_use_negative:
- When formula candidates are generated from a database search (NIST, GNPS, HMDB,
  PubChem) rather than by element-constrained mass decomposition — database entries
  are not bounded by SIRIUS decomp element caps
- When the analysis needs isotope-pattern or MS/MS scoring of candidate formulae,
  not just countability of the candidate space
- When formulae contain elements outside the SIRIUS decomp search space (e.g. metals)
  where the halogen/S/P caps are not the binding constraint
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Source of the element constraints ('SIRIUS decomp' dynamic-programming mass
    decomposition) whose feasibility limits are being verified against reference formulae
  repo: https://github.com/boecker-lab/sirius/
- name: Python
  role: Parse formula strings into element counts (molmass/Pyteomics or regex) and
    apply the constraint filter to compute the recoverable fraction
- name: MIST-CF
  role: Downstream formula scorer that relies on SIRIUS-generated candidate formulae,
    motivating the feasibility check of its candidate-generation constraints
  repo: github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:d450e6a48d32b2110c19a2723809273e7c5d209eaf32fae6029f91409b1eb043
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - MIST-CF
  - Python
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Element-count feasibility verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Parses reference molecular formulae into per-element counts and tests them against a candidate-generation constraint set (e.g. the SIRIUS 'SIRIUS decomp' element limits of max 5 S, max 3 P, and at most one of each halogen F/Cl/Br/I) to quantify what fraction of formulae a mass-decomposition pipeline can actually recover.

## When to use

Use when evaluating whether an element-constrained formula generation space (SIRIUS decomp-style limits) excludes a meaningful share of reference formulae from a spectral dataset such as NPLIB1 before benchmarking de novo formula predictors.

## When NOT to use

- When formula candidates are generated from a database search (NIST, GNPS, HMDB, PubChem) rather than by element-constrained mass decomposition — database entries are not bounded by SIRIUS decomp element caps
- When the analysis needs isotope-pattern or MS/MS scoring of candidate formulae, not just countability of the candidate space
- When formulae contain elements outside the SIRIUS decomp search space (e.g. metals) where the halogen/S/P caps are not the binding constraint

## Inputs

- Reference molecular formula list from NPLIB1 (canopus_train, Zenodo record 8151490) or any dataset of molecular formula strings
- Element constraint specification used by SIRIUS decomp (max 5 S, max 3 P, <=1 of each of F, Cl, Br, I)
- Python environment with a formula parser (molmass, Pyteomics, or regex-based parsing)

## Outputs

- recoverable_formulae.csv containing the recoverable fraction, per-formula element counts, and the excluded-species list with formulae and element counts
- Summary metric: fraction of NPLIB1 formulae recoverable under the constraints (~96%)

## How to apply

Download the reference dataset (e.g. NPLIB1/canopus_train from Zenodo record 8151490) and parse every reference molecular formula into element counts using a chemistry formula parser (molmass/Pyteomics or regex-based parsing). For each formula, evaluate the MIST-CF/SIRIUS mass-decomposition element constraints — max 5 S atoms, max 3 P atoms, and at most one of each of F, Cl, Br, I — to label it recoverable or excluded. Compute the fraction satisfying all constraints; for NPLIB1 this should reproduce the reported ~96% recovery (tolerance ±1%), implying only ~4% of reference formulae are unreachable by SIRIUS decomp candidate generation. Enumerate excluded species (e.g. iodixanol, with its formula and element counts) so the failure mode is inspectable, and write the recoverable fraction, counts, and excluded-species list to a CSV (e.g. recoverable_formulae.csv). The rationale is that MIST-CF delegates exact-mass decomposition to SIRIUS, so any formula violating these element caps can never appear among candidates regardless of the neural scorer's quality.

## Related tools

- **SIRIUS** (Source of the element constraints ('SIRIUS decomp' dynamic-programming mass decomposition) whose feasibility limits are being verified against reference formulae) — https://github.com/boecker-lab/sirius/
- **Python** (Parse formula strings into element counts (molmass/Pyteomics or regex) and apply the constraint filter to compute the recoverable fraction)
- **MIST-CF** (Downstream formula scorer that relies on SIRIUS-generated candidate formulae, motivating the feasibility check of its candidate-generation constraints) — github.com/samgoldman97/mist-cf

## Examples

```
wget https://zenodo.org/record/8151490/files/canopus_train.zip && unzip canopus_train.zip
```

## Evaluation signals

- Computed recoverable fraction matches the reported ~96% for NPLIB1 within ±1% tolerance
- recoverable_formulae.csv contains both the aggregate fraction and a non-empty excluded-species list (e.g. iodixanol) with full formulae and element counts
- Per-formula element counts round-trip: recombining counts reproduces the original formula string
- Excluded formulae each violate at least one constraint (S > 5, P > 3, or >1 of F/Cl/Br/I) — no excluded formula satisfies all caps

## Limitations

- The constraint set reflects SIRIUS decomp defaults used by MIST-CF; other tools (e.g. BUDDY) may use different element bounds, changing the recoverable fraction
- MIST-CF models were trained only for positive-mode data and the authors rely on SIRIUS for exact-mass decomposition, so feasibility under these caps bounds the whole pipeline regardless of the neural scorer
- A recoverable formula under element caps is necessary but not sufficient — recovery also depends on mass tolerance, adduct handling, and spectrum quality

## Evidence
