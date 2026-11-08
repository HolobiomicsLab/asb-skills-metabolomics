---
name: substructure-presence-vector-construction
description: Use when you have a molecule as a SMILES string and need the substructure
  presence representation (957 SMART-defined fragments) required to train or run inference
  with the substructure-to-structure transformer.
when_to_use_negative:
- Input is already a binary substructure label vector or integer index list (no re-computation
  needed)
- The goal is predicting substructures from 1H/13C NMR spectra rather than computing
  them from a known structure — that is the multitask model's job
- Molecules contain elements beyond C, N, and O as non-hydrogen atoms or exceed 19
  heavy atoms, outside the paper's validated scope
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Canonicalizes SMILES and computes the substructure presence vector via its
    substructure match functionality against the 957 SMART strings
- name: NMR2Struct
  role: Supplies the substructures_957.p SMART definitions, the token/config conventions
    (source_size=958), and the label hdf5 format
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:e67f306502f3d086c354ed65ecc4220f70cc940c2ad7960d9bd4aa1ea9be10a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# substructure-presence-vector-construction

## Summary

Constructs the 957-bit binary substructure presence vector for a molecule by canonicalizing its SMILES with RDKit and matching against 957 SMART substructure definitions, then compresses it to an integer index list for transformer embedding. This representation is the input format for the NMR2Struct substructure-to-structure model.

## When to use

Use when you have a molecule as a SMILES string and need the substructure presence representation (957 SMART-defined fragments) required to train or run inference with the substructure-to-structure transformer.

## When NOT to use

- Input is already a binary substructure label vector or integer index list (no re-computation needed)
- The goal is predicting substructures from 1H/13C NMR spectra rather than computing them from a known structure — that is the multitask model's job
- Molecules contain elements beyond C, N, and O as non-hydrogen atoms or exceed 19 heavy atoms, outside the paper's validated scope

## Inputs

- SMILES strings for molecules (canonicalized via RDKit)
- Pickle file of 957 SMART substructure definitions (example_configs/substructures_957.p)

## Outputs

- 957-bit binary substructure presence vector per molecule
- Compressed integer index list of present substructures (1–957) per molecule
- HDF5 label files with 'substructure_labels' dataset (one binary vector per row) for NMR2Struct training

## How to apply

Load the 957 SMART strings from NMR2Struct's example_configs/substructures_957.p and canonicalize each molecule's SMILES with RDKit's MolToSmiles. For each canonicalized molecule, use RDKit's substructure match functionality to compute a 957-bit binary vector indicating presence/absence of each fragment (fragments are ≤7 atoms). Compress the binary array into an integer array listing the index (1 to 957) of only the present (nonzero) substructures, since the transformer's embedding consumes the index list rather than the raw bit vector. When feeding the transformer, set model.model_args.source_size to number of substructures + 1 (958 for the 957 substructures) with src_pad_token = 0 so absent fragments map to padding. These vectors also become the 'substructure_labels' rows in hdf5 label files consumed by the NMR2Struct training config.

## Related tools

- **RDKit** (Canonicalizes SMILES and computes the substructure presence vector via its substructure match functionality against the 957 SMART strings)
- **NMR2Struct** (Supplies the substructures_957.p SMART definitions, the token/config conventions (source_size=958), and the label hdf5 format) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(Chem.MolToSmiles(Chem.MolFromSmiles(smi))); idx = [i+1 for i,p in enumerate(subs_957) if mol.HasSubstructMatch(p)]
```

## Evaluation signals

- Each molecule yields exactly a 957-element binary vector; after compression, all integer indices fall in the range 1–957
- Re-running RDKit substructure matching on the canonicalized SMILES reproduces the same vector (deterministic, order-independent of original SMILES form)
- Transformer source_size equals number of substructures + 1 = 958, matching the README token convention table
- Label file roundtrip: an hdf5 with dataset 'substructure_labels' has one row per molecule, each row the same length (957), values 0/1
- Sanity check on a known molecule: e.g. its characteristic functional-group SMART patterns appear as 1s at the expected indices

## Limitations

- The 957-substructure set does not encode stereochemistry; extending to stereospecific structure prediction requires expanding the substructure set with additional elements and larger chemical motifs
- Validated only on molecules with up to 19 heavy (non-hydrogen) atoms containing C, N, and O; substructure coverage outside this chemical space is untested

## Evidence

- [results] After canonicalizing all the SMILES strings using RDKit, the substructure vector for each molecule was computed using RDKit's substructure match functionality: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [results] an integer array was created that lists the index (1 to 957) of only the present (nonzero) substructures for each molecule.: "an integer array was created that lists the index (1 to 957) of only the present (nonzero) substructures for each molecule."
- [methods] the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used: "the example_configs/substructures_957.p file which contains the SMART strings for the 957 substructures used"
- [readme] These should be hdf5 files that contain one dataset called 'substructure_labels' where each row is a one-dimensional binary vector representing the substructures detected in each molecule.: "These should be hdf5 files that contain one dataset called 'substructure_labels' where each row is a one-dimensional binary vector representing the substructures detected in each molecule."
