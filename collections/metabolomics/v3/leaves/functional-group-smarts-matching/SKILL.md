---
name: functional-group-smarts-matching
description: Use when you have recommended SMILES structures (e.g. from Spec2Mol spectra-to-molecule
  translation) plus ground-truth test structures and need to check scaffold/functional-group
  presence via RDKit HasSubstructMatch.
when_to_use_negative:
- Input is not a pair of recommended/ground-truth SMILES sets from a spectrum-to-molecule
  model (e.g. raw spectra alone without decoder output)
- The evaluation goal is structural identity (exact-match or Tanimoto similarity on
  the full molecule) rather than scaffold/functional-group presence
- SMILES strings cannot be sanitized by RDKit (invalid chemistry) — HasSubstructMatch
  requires a valid Mol object
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3473
tools:
- name: RDKit
  role: Parses/canonicalizes SMILES, extracts Murcko scaffolds, and executes SMARTS
    HasSubstructMatch queries
  repo: https://www.rdkit.org/
- name: PyTorch
  role: Runs the Spec2Mol encoder-decoder that produced the recommended SMILES being
    evaluated
  repo: https://github.com/KavrakiLab/Spec2Mol
- name: Spec2Mol
  role: Source of the recommended SMILES structures that are checked for scaffold/functional-group
    presence
  repo: https://github.com/KavrakiLab/Spec2Mol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-023-00932-3
    title: An end-to-end deep learning framework for translating mass spectra to de-novo
      molecules
  source_content_hash: sha256:e2f743fc022877e3db0373f8d8b9e049eeee24c62cef346e7aa5096e8f26f17b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - RDKit
  - Spec2Mol
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# functional-group-smarts-matching

## Summary

Use RDKit SMARTS substructure queries to verify that Spec2Mol-recommended SMILES contain the ground-truth Bemis–Murcko scaffold and predefined functional groups, quantifying whether the model recovered key molecular substructures from MS/MS spectra.

## When to use

Use when you have recommended SMILES structures (e.g. from Spec2Mol spectra-to-molecule translation) plus ground-truth test structures and need to check scaffold/functional-group presence via RDKit HasSubstructMatch.

## When NOT to use

- Input is not a pair of recommended/ground-truth SMILES sets from a spectrum-to-molecule model (e.g. raw spectra alone without decoder output)
- The evaluation goal is structural identity (exact-match or Tanimoto similarity on the full molecule) rather than scaffold/functional-group presence
- SMILES strings cannot be sanitized by RDKit (invalid chemistry) — HasSubstructMatch requires a valid Mol object

## Inputs

- Recommended SMILES strings from Spec2Mol's decoder (one or more per test MS/MS spectrum)
- Ground-truth SMILES strings for the corresponding test compounds
- Predefined functional-group SMARTS patterns
- Optional: cached spectra embeddings from `predict_embs.py` (CSV m/z + intensity files for [M+H]+ and [M-H]- at 35% and 130% NCE)

## Outputs

- Per-compound boolean/presence table: ground-truth scaffold present in recommended SMILES (yes/no), each functional-group SMARTS present (yes/no)
- Aggregate counts and fractions: fraction of recommendations containing the ground-truth scaffold; fraction containing each functional group
- Canonicalized RDKit Mol objects for recommended and ground-truth structures

## How to apply

Load the Spec2Mol-recommended SMILES (from the spectra-to-SMILES decoder) alongside the corresponding ground-truth test structures, then parse and canonicalize all SMILES with RDKit, sanitizing molecules so they are substructure-searchable. Extract the Bemis–Murcko scaffold of each ground-truth molecule with RDKit's MurckoScaffold module. For each test compound, run RDKit HasSubstructMatch queries — one against the ground-truth scaffold and one against each predefined functional-group SMARTS pattern — on the recommended SMILES. Tabulate the fraction of recommendations that contain the ground-truth scaffold and each functional group, storing per-compound counts. Validation: confirm a nonzero substructure-recovery rate consistent with the paper's claim that 'Spec2Mol is able to identify the presence of key molecular substructures from its mass spectrum'. RDKit installation in the spec2mol conda environment (`conda install rdkit -c rdkit`) provides the required cheminformatics toolkit; the spectra encoder/decoder pipeline runs under PyTorch 1.6.0.

## Related tools

- **RDKit** (Parses/canonicalizes SMILES, extracts Murcko scaffolds, and executes SMARTS HasSubstructMatch queries) — https://www.rdkit.org/
- **PyTorch** (Runs the Spec2Mol encoder-decoder that produced the recommended SMILES being evaluated) — https://github.com/KavrakiLab/Spec2Mol
- **Spec2Mol** (Source of the recommended SMILES structures that are checked for scaffold/functional-group presence) — https://github.com/KavrakiLab/Spec2Mol

## Examples

```
from rdkit.Chem.Scaffolds import MurckoScaffold; scaff = MurckoScaffold.GetScaffoldForMol(MolFromSmiles(gt_smiles)); rec_mol = MolFromSmiles(rec_smiles); print(rec_mol.HasSubstructMatch(MolFromSmiles(scaff)))
```

## Evaluation signals

- All SMILES parse successfully in RDKit after sanitization (no parse failures or None Mol objects)
- Every ground-truth molecule yields a non-empty Bemis–Murcko scaffold via RDKit MurckoScaffold
- HasSubstructMatch queries execute without exceptions for both scaffold and functional-group SMARTS
- Substructure-recovery rate (fraction of recommendations containing the ground-truth scaffold) is nonzero and matches the paper's claim
- Per-compound counts sum to the number of evaluated test compounds

## Limitations

- The spectra encoder is trained on the commercial NIST Tandem Mass Spectral Library 2020; coverage/quality of scaffold recovery depends on that training distribution
- Scaffold and SMARTS presence are coarse-grained checks — they confirm substructure presence but not correct placement or full-molecule identity
- Requires SMILES that RDKit can sanitize; unparseable recommendations must be handled or excluded

## Evidence

- [abstract] Substructure check method: "Spec2Mol is able to identify the presence of key molecular substructures from its mass spectrum"
- [abstract] Decoder output format: "reconstructs SMILES sequences of the recommended chemical structures"
- [intro] RDKit dependency: "Processing of the chemical data is based on the RDKit software."
- [methods] RDKit installation command: "conda install rdkit -c rdkit"
- [readme] Input spectra format and collision energies: "pos_low_file`: precursor [M+H]+, energy 35% NCE (Normalized Collision Energy)"
