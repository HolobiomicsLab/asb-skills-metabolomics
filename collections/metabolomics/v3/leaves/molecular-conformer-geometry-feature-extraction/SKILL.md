---
name: molecular-conformer-geometry-feature-extraction
description: Use when 3D molecular conformations (e.g. [max_atom_num, 21] arrays of
  xyz + 18 one-hot atom features with a covalent bond graph) must be turned into pose-independent
  encoder inputs, or when verifying E(3)/SE(3) invariance of a molecular geometry
  layer.
when_to_use_negative:
- Input molecules have no 3D conformations and only 2D structures are available (generate
  ETKDGv3 conformers first, e.g. via molecules_to_records, or the geometry channels
  are undefined).
- The downstream task must distinguish enantiomers while using the plain E(3) encoder
  mode — SE(3) chirality mode is required for chiral separation.
- The goal is spectrum decoding rather than geometric encoding (e.g. the bidirectional
  MS/MS head with precursor mask), which operates on the encoder output, not on raw
  conformer geometry.
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_2258
tools:
- name: molnetpack
  role: PyPI package providing the MolConv layer (molnetpack.molconv) and preprocessing
    utilities (conformation_array, bond_graph_array) used to build geometry features
  repo: https://github.com/JosieHong/3DMolMS
- name: PyTorch
  role: Backend for instantiating MolConv layers and running the embedding/invariance
    transformations
- name: NumPy
  role: Constructing the [max_atom_num, 21] conformer arrays and computing float64
    max-deviation comparisons between transformed embeddings
- name: RDKit
  role: Dependency for generating 3D conformations (ETKDGv3 conformers) and covalent
    bond graphs prior to feature extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:67b8924b0fba588b9a5a88740c0ba45c674738652b5c6047cb9bafdf84a7a4e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - NumPy
  - PyTorch
  - Python+
  - RDKit
  - molnetpack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# molecular-conformer-geometry-feature-extraction

## Summary

Extracts pose-independent geometric features from 3D molecular conformations using the 3DMolMS MolConv elemental operation, which aggregates k-nearest-neighbor distances and inner-product angle terms over relative displacements rather than absolute coordinates.

## When to use

Use when 3D molecular conformations (e.g. [max_atom_num, 21] arrays of xyz + 18 one-hot atom features with a covalent bond graph) must be turned into pose-independent encoder inputs, or when verifying E(3)/SE(3) invariance of a molecular geometry layer.

## When NOT to use

- Input molecules have no 3D conformations and only 2D structures are available (generate ETKDGv3 conformers first, e.g. via molecules_to_records, or the geometry channels are undefined).
- The downstream task must distinguish enantiomers while using the plain E(3) encoder mode — SE(3) chirality mode is required for chiral separation.
- The goal is spectrum decoding rather than geometric encoding (e.g. the bidirectional MS/MS head with precursor mask), which operates on the encoder output, not on raw conformer geometry.

## Inputs

- 3D conformer point set as a [max_atom_num, 21] array (3 xyz coordinates + 18 one-hot atom features)
- Covalent bond graph supplied by preprocessing (data_utils.bond_graph_array), with self-pointing padding for unused slots
- Molecule records from molnetpack.molecules_to_records / conformation_array (ETKDGv3 conformers from SDF or ChEMBL sources)

## Outputs

- MolConv point-embedding tensor for each atomic point (aggregate of distance-filtered W1, angle-filtered W2, and neighbor features)
- Per-transformation invariance report (molconv_invariance_report.json) with max absolute embedding deviations and pass/fail flags
- Saved molecular embeddings via molnet_engine.save_features() (ids plus feature matrix)

## How to apply

Instantiate a MolConv layer from molnetpack.molconv with random weights (invariance is architectural, so no trained checkpoint is needed). Represent each molecule as a [max_atom_num, 21] array (3 xyz coordinates + 18 one-hot features) with a covalent-style fixed neighbor graph and self-pointing padding for unused slots. At each layer, update the point representation xi by summing over its k-nearest neighbors: apply the distance filter W1 to the pairwise distance and the direction filter W2 to the inner-product angle term ⟨xj−xi, xl−xi⟩ computed from displacements centered on the real-atom centroid, both multiplied element-wise with the neighbor feature xj (Eq. 3). Recompute geometry from relative positions after each perturbation (atom-list permutation, random translation, random rotation, mirror reflection) and record the maximum absolute embedding difference in float64. In SE(3) chirality mode, expect sign-flip sensitivity under reflection; the plain E(3) mode is provably unable to model enantiomer differences.

## Related tools

- **molnetpack** (PyPI package providing the MolConv layer (molnetpack.molconv) and preprocessing utilities (conformation_array, bond_graph_array) used to build geometry features) — https://github.com/JosieHong/3DMolMS
- **PyTorch** (Backend for instantiating MolConv layers and running the embedding/invariance transformations)
- **NumPy** (Constructing the [max_atom_num, 21] conformer arrays and computing float64 max-deviation comparisons between transformed embeddings)
- **RDKit** (Dependency for generating 3D conformations (ETKDGv3 conformers) and covalent bond graphs prior to feature extraction)

## Examples

```
from molnetpack import MolNet; molnet_engine = MolNet(device, seed=42); molnet_engine.load_data('./examples/demo_input.csv'); ids, features = molnet_engine.save_features()
```

## Evaluation signals

- Maximum absolute embedding deviation is floating-point-exact (≈0 in float64) under atom-list permutation, random translation, and random rotation, matching the paper's assertion of full E(3) + permutation invariance for MolConv.
- Under mirror reflection with chirality: true (SE(3) mode), embeddings flip sensitivity as expected; with the plain E(3) encoder, both enantiomers necessarily yield one identical value.
- molnetpack's own suites pass: pytest tests/test_encoder_invariance.py tests/test_se3_invariance.py -q
- Geometry is recomputed from relative displacements ⟨xj−xi, xl−xi⟩ centered on the real-atom centroid, so embeddings do not change when the molecule's pose changes.
- Feature matrix shape from save_features() matches the number of loaded molecules and the encoder embedding dimension.

## Limitations

- The earlier MolConv1 layer computed geometry from absolute atom positions and was not translation-invariant; it was removed in v1.4.0, so only the relative-geometry MolConv should be used.
- An E(3) encoder is provably unable to model enantiomer differences; chiral tasks require the SE(3) chirality mode.
- Input constraints: atom number ≤ 300 and atom types limited to C, O, N, H, P, S, F, Cl, B, Br, I; unsupported formats are automatically excluded during loading.

## Evidence

- [methods] MolConv achieves pose-independence by describing geometry through relative positions and angles rather than absolute coordinates: [evidence span withheld — anchor status: not_found]
- [methods] At each layer, the point representation xi is updated by summing over its k-nearest neighbors: [evidence span withheld — anchor status: not_found]
- [methods] MolConv1 removed in v1.4.0: "An earlier version of this layer (`MolConv1`) computed geometry from absolute atom positions and was removed in v1.4.0 because it was not translation-invariant"
- [methods] E(3) cannot model enantiomers: "An E(3) encoder is **provably unable** to model enantiomer differences (it must output one value for both), so SE(3) is *required* for chiral separation"
- [readme] Representation transfers to RT and CCS: "The molecular representation it learns transfers to related tasks such as retention time (RT) and collision cross section (CCS) prediction."
