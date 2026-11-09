---
name: rdkit-substructure-search
description: Use when you have Spec2Mol-recommended SMILES and ground-truth structures
  from an MS/MS test set and need to quantify recovery of key substructures (scaffolds,
  functional groups) in the recommendations.
when_to_use_negative:
- When the evaluation target is overall molecular similarity between recommended and
  ground-truth structures rather than presence/absence of specific substructures.
- When inputs are raw MS/MS spectra (m/z, intensity pairs) rather than SMILES strings
  — this skill operates post-decoder, on generated structures.
- When ground-truth structures are unavailable for the test compounds, since no scaffold
  or functional-group reference exists to query against.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: RDKit
  role: Canonicalize SMILES, extract Bemis-Murcko scaffolds, and run HasSubstructMatch
    queries for scaffolds and functional-group SMARTS
  repo: https://www.rdkit.org/
- name: PyTorch
  role: Framework underlying Spec2Mol, whose recommended SMILES serve as the query
    structures for substructure checks
  repo: https://github.com/KavrakiLab/Spec2Mol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-023-00932-3
    title: An end-to-end deep learning framework for translating mass spectra to de-novo
      molecules
  source_content_hash: sha256:7a1facaa927b80a6c14240a40cf3e675c52303bf9d0fa58b55de81927f25c73d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# rdkit-substructure-search

## Summary

Use RDKit to test whether Spec2Mol's recommended SMILES reproduce key molecular substructures (Bemis-Murcko scaffolds and functional-group SMARTS patterns) present in ground-truth test structures, validating structure recommendations at the substructure level rather than by overall similarity alone.

## When to use

Use when you have Spec2Mol-recommended SMILES and ground-truth structures from an MS/MS test set and need to quantify recovery of key substructures (scaffolds, functional groups) in the recommendations.

## When NOT to use

- When the evaluation target is overall molecular similarity between recommended and ground-truth structures rather than presence/absence of specific substructures.
- When inputs are raw MS/MS spectra (m/z, intensity pairs) rather than SMILES strings — this skill operates post-decoder, on generated structures.
- When ground-truth structures are unavailable for the test compounds, since no scaffold or functional-group reference exists to query against.

## Inputs

- Spec2Mol-recommended SMILES strings from the similarity-evaluation stage (decoder output for MS/MS spectra)
- Ground-truth test molecular structures (SMILES)
- Predefined functional-group SMARTS patterns
- RDKit environment (conda install rdkit -c rdkit)

## Outputs

- Per-compound counts of scaffold and functional-group matches in the recommended structures
- Aggregate fraction of recommendations containing the ground-truth scaffold
- Aggregate recovery rate for each functional group
- Validation report showing nonzero substructure recovery

## How to apply

Parse and canonicalize both the Spec2Mol-recommended SMILES (decoder output for the [M+H]+/[M-H]- spectra at 35%/130% NCE) and the corresponding ground-truth test structures with RDKit, sanitizing each molecule so substructure queries are well-defined. Extract the Bemis-Murcko scaffold of each ground-truth molecule with RDKit's MurckoScaffold module, then run HasSubstructMatch queries against the recommended SMILES to check scaffold presence and the presence of predefined functional-group SMARTS patterns. Tabulate, per test compound, the fraction of recommendations containing the ground-truth scaffold and each functional group. Validate by confirming a nonzero substructure-recovery rate, consistent with the paper's claim that Spec2Mol identifies the presence of key molecular substructures from the mass spectrum.

## Related tools

- **RDKit** (Canonicalize SMILES, extract Bemis-Murcko scaffolds, and run HasSubstructMatch queries for scaffolds and functional-group SMARTS) — https://www.rdkit.org/
- **PyTorch** (Framework underlying Spec2Mol, whose recommended SMILES serve as the query structures for substructure checks) — https://github.com/KavrakiLab/Spec2Mol

## Examples

```
python -c "from rdkit import Chem; from rdkit.Chem.Scaffolds import MurckoScaffold; q = Chem.MolFromSmiles(rec_smiles); ref = Chem.MolFromSmiles(true_smiles); scaffold = MurckoScaffold.GetScaffoldForMol(ref); print(q.HasSubstructMatch(scaffold))"
```

## Evaluation signals

- Nonzero fraction of recommended SMILES containing the ground-truth Bemis-Murcko scaffold (HasSubstructMatch returns True for at least some compounds).
- Per-compound table is populated for every test compound with scaffold and functional-group counts.
- Recovered substructure counts are consistent with the paper's claim that Spec2Mol identifies key molecular substructures from the mass spectrum.
- All SMILES canonicalized and sanitized before matching (RDKit does not raise sanitization errors).

## Limitations

- Substructure presence is a binary, qualitative signal — it does not capture stereochemical or full-structure fidelity of the recommendation.
- The NIST Tandem Mass Spectral Library 2020 used for encoder training is commercial, so evaluation depends on test structures not seen during training or present in the reference database.
- Functional-group SMARTS patterns must be predefined by the practitioner; the choice of patterns directly affects recovery rates.

## Evidence

- [abstract] Substructure recovery claim: "Spec2Mol is able to identify the presence of key molecular substructures from its mass spectrum"
- [abstract] SMILES output of decoder: "reconstructs SMILES sequences of the recommended chemical structures"
- [intro] RDKit used for chemical processing: "Processing of the chemical data is based on the RDKit software."
- [methods] RDKit install command: "conda install rdkit -c rdkit"
- [intro] PyTorch underlying Spec2Mol: "The implementation of the Spec2Mol architecture is based on the Pytorch library."
- [methods] Conda environment setup: [evidence span withheld — anchor status: not_found]
