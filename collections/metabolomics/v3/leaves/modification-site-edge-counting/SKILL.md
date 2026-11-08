---
name: modification-site-edge-counting
description: Use when building a benchmark of structurally related compound pairs
  from reference MS/MS libraries where each pair must differ by exactly one modification
  site, and you need to verify single-edge boundaries with RDKit substructure matching.
when_to_use_negative:
- When compound pairs differ by more than one edge between the matched substructure
  and the remainder, i.e. multiple simultaneous modification sites
- When precursor mass exceeds 2000 Da or the mass difference exceeds 50% of the smaller
  compound's precursor mass
- When the two compounds do not share the same M+H adduct or were acquired under non-matched
  instrument conditions
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Convert SMILES to Mol objects and perform GetSubstructMatch to identify the
    common scaffold and count boundary edges between matched substructure and remainder
- name: ModiFinder
  role: Downstream consumer of the benchmark pair table for evaluating modification
    site localization from MS/MS alignment
  repo: github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MS/MS library search
  role: Source of the reference spectra and metadata used to assemble the analog pair
    benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:d372c4dfac06b23f08ed4122d4d16f84481e7bc0ee041eb663e60c212e7c94fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS/MS library search
  - ModiFinder
  - RDKit
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# modification-site-edge-counting

## Summary

Enumerate candidate analog pairs from public MS/MS libraries and count the graph edges between the matched substructure and the remainder of the heavier compound, retaining only pairs with exactly one such edge so that a single structural modification site is defined.

## When to use

Use when building a benchmark of structurally related compound pairs from reference MS/MS libraries where each pair must differ by exactly one modification site, and you need to verify single-edge boundaries with RDKit substructure matching.

## When NOT to use

- When compound pairs differ by more than one edge between the matched substructure and the remainder, i.e. multiple simultaneous modification sites
- When precursor mass exceeds 2000 Da or the mass difference exceeds 50% of the smaller compound's precursor mass
- When the two compounds do not share the same M+H adduct or were acquired under non-matched instrument conditions

## Inputs

- MS/MS spectra and SMILES strings from public reference libraries GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, and BERKELEY-LAB
- Within-library compound pairs enumerated from the above libraries

## Outputs

- Benchmark pair table of 12,909 analog pairs, each with exactly one modification site, annotated with source library
- RDKit Mol objects with the matched substructure atom set identified for each accepted pair

## How to apply

Retrieve MS/MS spectra and 2D SMILES from the four GNPS reference libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) and enumerate every within-library compound pair. Apply the eligibility filters described in the article: precursor mass must be less than 2000 Da, the precursor mass difference must be less than 50% of the smaller compound's mass, and both members must share the same M+H adduct and matched instrument conditions. Convert both SMILES strings to RDKit Mol objects, then call GetSubstructMatch using the smaller compound as the query on the heavier compound to identify the atoms belonging to the common scaffold. Count the bonds in the heavier molecule that connect an atom inside the matched substructure to an atom outside it; discard any pair with more than one such edge, retaining only those with exactly one modification site. The surviving pairs, annotated with source library, should total 12,909 analog pairs for this benchmark.

## Related tools

- **RDKit** (Convert SMILES to Mol objects and perform GetSubstructMatch to identify the common scaffold and count boundary edges between matched substructure and remainder)
- **ModiFinder** (Downstream consumer of the benchmark pair table for evaluating modification site localization from MS/MS alignment) — github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MS/MS library search** (Source of the reference spectra and metadata used to assemble the analog pair benchmark)

## Examples

```
rdmol_heavy = Chem.MolFromSmiles(smiles_heavy); matches = rdmol_heavy.GetSubstructMatch(Chem.MolFromSmiles(smiles_small))
```

## Evaluation signals

- The filtered pair set must total exactly 12,909 analog pairs, matching the reported benchmark size
- Every accepted pair must have exactly one edge connecting the matched substructure to the rest of the molecule, i.e. exactly one modification site
- Each accepted pair must satisfy precursor mass < 2000 Da and precursor mass difference < 50% of the smaller compound's precursor mass
- Both members of each pair must share the same M+H adduct and matched instrument conditions
- The heavier compound must contain the smaller compound as a substructure (GetSubstructMatch succeeds)

## Limitations

- Pairs with more than one boundary edge between the matched substructure and the remainder are discarded, so multi-site modifications are excluded
- Only M+H adducts under matched instrument conditions are considered; other adduct types are outside the benchmark
- Precursor mass limit of 2000 Da and 50% mass-difference cutoff restrict the scope to small-molecule analogs

## Evidence

- [methods] substructure edge counting: [evidence span withheld — anchor status: not_found]
- [readme] tool description: [evidence span withheld — anchor status: not_found]
