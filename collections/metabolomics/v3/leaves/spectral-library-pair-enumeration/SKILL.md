---
name: spectral-library-pair-enumeration
description: Use when you need to build a structurally grounded analog benchmark from
  public MS/MS library entries (spectra + SMILES) for modification-site-localization
  or analog-search evaluation.
when_to_use_negative:
- When the goal is analog search over an unknown sample rather than constructing a
  ground-truth benchmark of structurally related library pairs.
- When compound pairs lack same M+H adducts or differ by more than one structural
  modification — these are explicitly filtered out by the pair eligibility criteria.
- When the input is a single spectrum or a single compound rather than a collection
  of library entries that can be paired within-library.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Downstream evaluation target whose benchmark pair set is produced by this
    enumeration skill.
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: RDKit
  role: Used to convert SMILES to molecule objects and run GetSubstructMatch to verify
    substructure relationships and count modification-site edges.
- name: MS/MS library search
  role: Source of the public reference MS/MS libraries that supply the spectra and
    SMILES to be paired.
- name: Analog library search
  role: Related analog-identification capability that motivates building the pair
    benchmark.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:3cd1e0acf68fbbc5947204be0b052a93a9fbcf01290ca4692537285c57c0abf1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - Analog library search
  - MS/MS library search
  - ModiFinder
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# spectral-library-pair-enumeration

## Summary

Enumerates every within-library compound pair from public GNPS MS/MS reference libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) and applies structural/mass eligibility filters to assemble a benchmark of 12,909 single-modification analog pairs for evaluating ModiFinder's modification site localization.

## When to use

Use when you need to build a structurally grounded analog benchmark from public MS/MS library entries (spectra + SMILES) for modification-site-localization or analog-search evaluation.

## When NOT to use

- When the goal is analog search over an unknown sample rather than constructing a ground-truth benchmark of structurally related library pairs.
- When compound pairs lack same M+H adducts or differ by more than one structural modification — these are explicitly filtered out by the pair eligibility criteria.
- When the input is a single spectrum or a single compound rather than a collection of library entries that can be paired within-library.

## Inputs

- MS/MS spectra from GNPS-MSMLS
- MS/MS spectra from GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE
- MS/MS spectra from GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE
- MS/MS spectra from BERKELEY-LAB
- 2D structures (SMILES) corresponding to each library entry
- Precursor mass and adduct annotation per library entry

## Outputs

- Benchmark pair table of 12,909 analog pairs annotated with source library
- Per-pair modification-site count (1 for retained pairs)

## How to apply

Retrieve the MS/MS spectra and 2D structures (SMILES) from the four public reference libraries, then enumerate every possible within-library compound pair. Apply the paper's eligibility filters: precursor mass < 2000 Da, precursor mass difference < 50% of the smaller compound's precursor mass, and both members sharing the same M+H adduct. For each surviving pair, convert both SMILES strings to RDKit molecule objects and call GetSubstructMatch with the smaller compound as the query on the heavier compound to confirm a substructure relationship; then count the edges between atoms in the matched substructure set and atoms outside it as the number of modification sites, discarding any pair with more than one edge. The rationale is that only single-modification analog pairs can serve as ground-truth test cases for ModiFinder's site-localization scoring, and benchmark assembly is explicitly described as a multi-step process over the four GNPS libraries.

## Related tools

- **ModiFinder** (Downstream evaluation target whose benchmark pair set is produced by this enumeration skill.) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **RDKit** (Used to convert SMILES to molecule objects and run GetSubstructMatch to verify substructure relationships and count modification-site edges.)
- **MS/MS library search** (Source of the public reference MS/MS libraries that supply the spectra and SMILES to be paired.)
- **Analog library search** (Related analog-identification capability that motivates building the pair benchmark.)

## Examples

```
from rdkit import Chem; Chem.MolFromSmiles(smaller_smiles) and Chem.MolFromSmiles(heavier_smiles) via GetSubstructMatch to confirm single-modification analog pairs
```

## Evaluation signals

- The filtered pair set totals exactly 12,909 analog pairs, matching the reported benchmark size.
- Every retained pair has both members sharing the same M+H adduct and a precursor mass below 2000 Da.
- Every retained pair shows a valid RDKit GetSubstructMatch between the smaller compound (query) and the heavier compound.
- Every retained pair has exactly one edge between atoms in the matched substructure and atoms outside it (single modification site).
- Each pair in the output table is annotated with its source library (one of the four GNPS/Berkeley libraries).

## Limitations

- Pairs with precursor mass difference ≥ 50% of the smaller compound's precursor mass are excluded, restricting coverage to close analogs.
- Pairs whose SMILES yield more than one edge between matched and unmatched atoms are discarded, so multi-site modifications are not represented in the benchmark.
- The enumeration is within-library only; cross-library compound pairs are not considered by this pipeline.

## Evidence

- [results] The aggregated benchmark built from the four reference MS/MS libraries contains 12,909 analog pairs with M+H adducts differing by a single structural modification.: [evidence span withheld — anchor status: not_found]
- [readme] ModiFinder tool role: [evidence span withheld — anchor status: not_found]
