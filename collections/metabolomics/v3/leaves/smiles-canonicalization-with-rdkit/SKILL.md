---
name: smiles-canonicalization-with-rdkit
description: Use when raw (non-canonical) SMILES strings must be converted to one
  consistent canonical form before computing substructure presence vectors, building
  the SMILES alphabet, or comparing predicted vs. true structures.
when_to_use_negative:
- Input is already canonical SMILES or an existing precomputed label file — re-canonicalizing
  may break correspondence with the NMR2Struct dataset splits
- Molecules with more than 19 heavy atoms — outside the framework's demonstrated scope,
  so the downstream substructure vectors will not match the trained model
- The target is stereochemistry-aware structure prediction — the 957-substructure
  set does not encode stereochemical motifs
tools:
- name: RDKit
  role: Parses raw SMILES, produces the canonical SMILES via Chem.MolToSmiles, and
    performs the substructure match (HasSubstructMatch) against the 957 SMART patterns
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: NMR2Struct
  role: Consumes the canonical-SMILES-derived substructure labels and SMILES alphabet
    for training the substructure-to-structure transformer and multitask model
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:d352557022c2827c54bab4c1b9ecea5a0db3580bc7e64048be63e2c335f84b3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# SMILES canonicalization with RDKit

## Summary

Canonicalize every molecule's SMILES string with RDKit so that substructure matching against the 957 SMART patterns, tokenization, and train/validation/test consistency in NMR2Struct are performed on a single normalized representation.

## When to use

Use when raw (non-canonical) SMILES strings must be converted to one consistent canonical form before computing substructure presence vectors, building the SMILES alphabet, or comparing predicted vs. true structures.

## When NOT to use

- Input is already canonical SMILES or an existing precomputed label file — re-canonicalizing may break correspondence with the NMR2Struct dataset splits
- Molecules with more than 19 heavy atoms — outside the framework's demonstrated scope, so the downstream substructure vectors will not match the trained model
- The target is stereochemistry-aware structure prediction — the 957-substructure set does not encode stereochemical motifs

## Inputs

- SMILES strings for each molecule (from the dataset in Zenodo record 13892026 or the numpy binary-encoded smiles_file)
- 957 substructure SMART strings in example_configs/substructures_957.p (pickle)
- NMR2Struct YAML configuration file with data.spectra_file / data.label_file / data.smiles_file lists

## Outputs

- Canonicalized SMILES strings (one per molecule) suitable for regex tokenization and alphabet construction
- Per-molecule 957-bit binary substructure vectors stored as the 'substructure_labels' dataset in an HDF5 file
- Canonical SMILES used for exact-match evaluation of generated predictions

## How to apply

Parse each SMILES string into an RDKit Mol object with Chem.MolFromSmiles and re-emit it with Chem.MolToSmiles to obtain the canonical form. After canonicalizing all the SMILES strings using RDKit, run RDKit's HasSubstructMatch against the 957 SMART strings loaded from example_configs/substructures_957.p (pickle) to build each molecule's 957-bit presence/absence vector, which is stored as an HDF5 dataset named 'substructure_labels'. Canonical SMILES is also the comparison standard for generated predictions: outputs from top-k sampling (k=5, 15 predictions per input) are evaluated by canonical SMILES match. Rows in the label file must correspond element-wise to the spectra_file and smiles_file entries declared in the YAML config.

## Related tools

- **RDKit** (Parses raw SMILES, produces the canonical SMILES via Chem.MolToSmiles, and performs the substructure match (HasSubstructMatch) against the 957 SMART patterns) — https://github.com/MarklandGroup/NMR2Struct
- **NMR2Struct** (Consumes the canonical-SMILES-derived substructure labels and SMILES alphabet for training the substructure-to-structure transformer and multitask model) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
from rdkit import Chem; can_smi = Chem.MolToSmiles(Chem.MolFromSmiles(smiles), canonical=True)
```

## Evaluation signals

- The saved HDF5 file contains a dataset named 'substructure_labels' with exactly 957 binary values per row
- Rows are keyed to the same molecule identifiers/order as the spectra_file and smiles_file lists (element-wise correspondence in the YAML config)
- Each 957-bit vector, when expanded from its compact integer index list (indices 1–957 of present substructures), matches the HasSubstructMatch output against substructures_957.p
- Re-canonicalizing a canonicalized string is idempotent (Chem.MolToSmiles output is unchanged)
- Tokenization of the canonical SMILES with the regex tokenizer yields tokens drawn from the saved alphabet (e.g., example_configs/alphabet.npy)

## Limitations

- The 957-substructure set contains no stereochemical information, so canonicalized structures feed a model that does not predict stereochemistry
- Substructure vectors were computed for molecules with C, N, and O as non-hydrogen atoms (SpectraBase ∼143k set plus ∼3M GDB-17 samples); other element sets are out of scope
- SMILES stored in the numpy smiles_file are binary-encoded and require x.decode('utf-8') before parsing

## Evidence

- [results] After canonicalizing all the SMILES strings using RDKit, the substructure vector for each molecule was computed using RDKit’s substructure match functionality: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [results] an integer array was created that lists the index (1 to 957) of only the present (nonzero) substructures for each molecule.: "an integer array was created that lists the index (1 to 957) of only the present (nonzero) substructures for each molecule."
- [readme] These should be hdf5 files that contain one dataset called 'substructure_labels' where each row is a one-dimensional binary vector: "These should be hdf5 files that contain one dataset called 'substructure_labels' where each row is a one-dimensional binary vector representing the substructures detected in each molecule."
- [methods] the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used: "the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used"
- [results] the canonicalized SMILES were tokenized using a regular expression,32 and an alphabet was determined from all unique tokens across all SMILES: "the canonicalized SMILES were tokenized using a regular expression,32 and an alphabet was determined from all unique tokens across all SMILES"
