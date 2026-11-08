---
name: dataset-split-keyed-identifier-mapping
description: Use when generating per-molecule feature or label arrays (e.g., 957-bit
  substructure vectors from SMILES) that must be keyed and ordered to match NMR2Struct
  HDF5 label/spectra files and the train/validation/test splitting dictionary.
when_to_use_negative:
- When the labels already exist as NMR2Struct-compatible HDF5 'substructure_labels'
  files — do not recompute or re-key them
- When the target representation is the compressed integer array of present-substructure
  indices (1 to 957) used for transformer source tokens — that is a downstream compaction
  of this binary vector, not its replacement
- When running inference or model training only — this skill prepares the label data,
  it does not generate predictions or process spectra
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: RDKit
  role: Canonicalizes SMILES and performs the substructure matching (HasSubstructMatch)
    against the 957 SMART patterns to build each binary vector
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: NMR2Struct
  role: Defines the consumed label file format (HDF5 dataset 'substructure_labels',
    one binary vector per molecule) and the splitting-file convention; provides substructures_957.p
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:ce9b57dfcc197d07965c194a573332fe82a54b1a54d86e0e3be76874bdb64951
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# dataset-split-keyed-identifier-mapping

## Summary

Builds per-molecule derived arrays (e.g., 957-substructure presence/absence vectors from RDKit SMART matching) whose rows are keyed by molecule identifier so they row-align with NMR2Struct's spectra, label, and SMILES files and can be partitioned by the train/val/test split dictionary.

## When to use

Use when generating per-molecule feature or label arrays (e.g., 957-bit substructure vectors from SMILES) that must be keyed and ordered to match NMR2Struct HDF5 label/spectra files and the train/validation/test splitting dictionary.

## When NOT to use

- When the labels already exist as NMR2Struct-compatible HDF5 'substructure_labels' files — do not recompute or re-key them
- When the target representation is the compressed integer array of present-substructure indices (1 to 957) used for transformer source tokens — that is a downstream compaction of this binary vector, not its replacement
- When running inference or model training only — this skill prepares the label data, it does not generate predictions or process spectra

## Inputs

- SMILES strings for each molecule in the training/validation/test sets (from Zenodo record 13892026)
- Pickle file example_configs/substructures_957.p containing SMART strings for the 957 substructures
- Optional splitting dictionary with three fields of indices (training, validation, test)
- NMR2Struct YAML config with data.spectra_file / data.label_file / data.smiles_file lists of equal length

## Outputs

- HDF5 file containing one dataset named 'substructure_labels' with one row per molecule, each a 957-value binary presence/absence vector keyed by molecule identifier
- Rows ordered/addressable by the indices in the train/val/test splitting dictionary

## How to apply

Load the 957 SMART strings from example_configs/substructures_957.p with pickle and the SMILES strings for each molecule from the dataset (Zenodo record 13892026). Canonicalize each SMILES with RDKit, then call HasSubstructMatch against all 957 SMART patterns to produce a 957-bit binary presence/absence vector per molecule. Assemble the vectors row-per-molecule in the same identifier order as the spectra and SMILES files, since NMR2Struct requires element-wise correspondence between the spectra_file, label_file, and smiles_file lists. Write the result as an HDF5 dataset named 'substructure_labels' where each row is a one-dimensional binary vector, matching the label file format read by NMR2Struct. If a splitting dictionary is used instead of train_size/val_size/test_size fractions, ensure the indices it defines (training, validation, test fields) index into rows of this array so each split resolves to the correct molecules.

## Related tools

- **RDKit** (Canonicalizes SMILES and performs the substructure matching (HasSubstructMatch) against the 957 SMART patterns to build each binary vector) — https://github.com/MarklandGroup/NMR2Struct
- **NMR2Struct** (Defines the consumed label file format (HDF5 dataset 'substructure_labels', one binary vector per molecule) and the splitting-file convention; provides substructures_957.p) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
mol = Chem.MolFromSmiles(Chem.CanonSmiles(smi)); vec = [int(mol.HasSubstructMatch(p)) for p in patts_957]
```

## Evaluation signals

- Saved HDF5 file contains exactly one dataset named 'substructure_labels'
- Each row has 957 binary (0/1) values, one per substructure SMART pattern
- Row count and ordering are consistent with the corresponding SMILES and spectra files (element-wise correspondence of the spectra_file/label_file/smiles_file lists)
- All indices in the training/validation/test splitting dictionary fall within the row count of the array
- Spot-check a molecule: recomputing HasSubstructMatch on its canonicalized SMILES reproduces its stored row

## Limitations

- The 957-substructure set defines the label space; molecules with elements or motifs outside this set are only partially described (the paper notes stereochemistry and larger motifs are not covered)
- Identifier alignment is order-based: if rows are shuffled or keyed differently from the spectra/SMILES files, the split dictionary will silently map to wrong molecules
- Software and package versions are not stated in the README; verify RDKit behavior for SMART matching on edge-case valences

## Evidence

- [results] After canonicalizing all the SMILES strings using RDKit, the substructure vector for each molecule was computed using RDKit’s substructure match functionality against the set of 957 substructures: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality against the set of 957 substructures"
- [readme] label files contain the dataset 'substructure_labels' with one binary vector per molecule: "label: These should be hdf5 files that contain one dataset called 'substructure_labels' where each row is a one-dimensional binary vector representing the substructures detected in each molecule."
- [readme] splitting file is a dictionary of training, validation, and testing indices: "The splitting file should be a dictionary of three fields which specify the indices that should go into the training, validation, and testing set, respectively."
