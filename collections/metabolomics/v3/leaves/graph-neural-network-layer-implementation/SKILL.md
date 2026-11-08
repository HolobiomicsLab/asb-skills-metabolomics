---
name: graph-neural-network-layer-implementation
description: Use when building or validating a GNN layer for 3D molecular point sets
  (xyz coordinates + atom one-hot features) where embeddings must be provably invariant
  to atom ordering, translation, and rotation — e.g. as an encoder for MS/MS, RT,
  or CCS prediction.
when_to_use_negative:
- The molecule has no 3D conformer (input is only a 2D graph or SMILES) — MolConv's
  geometry channels require xyz coordinates, so generate ETKDGv3 conformers first
  via molecules_to_records/sdf2pkl
- The task is chiral/enantiomer discrimination with a purely E(3)-invariant encoder
  — an E(3) encoder provably cannot distinguish enantiomers, so the SE(3) chirality
  mode is required
- Geometry is computed from absolute atom positions (the deprecated MolConv1 formulation)
  — this breaks translation invariance and was removed in v1.4.0
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: molnetpack
  role: Provides the MolConv layer implementation (molnetpack.molconv) used to build
    and test the GNN layer
  repo: https://pypi.org/project/molnetpack/
- name: 3DMolMS
  role: The 3DMolMS model whose encoder contains the MolConv layer being implemented;
    installed via molnetpack or from source
  repo: https://github.com/JosieHong/3DMolMS
- name: PyTorch
  role: Backend for instantiating the MolConv layer and running the transformed inputs
    (must be installed separately)
  repo: https://pytorch.org/get-started/locally/
- name: NumPy
  role: Generates random point sets and records per-transformation deviation values
    in float64 for the invariance report
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:cd8eaec1231acdd88df92fdc0ed0a3b9efd61d2b3cf37152c270aa6d04e78a61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 3DMolMS
  - NumPy
  - PyTorch
  - Python+
  - molnetpack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# graph-neural-network-layer-implementation

## Summary

Implements the MolConv layer from 3DMolMS (v1.4.0, molnetpack): a geometry-aware GNN message-passing layer that aggregates over the covalent bond graph using distance and direction filters on relative displacements, giving embeddings invariant to atom permutation, translation, and rotation. This skill covers both constructing the layer and numerically verifying its architectural invariances.

## When to use

Use when building or validating a GNN layer for 3D molecular point sets (xyz coordinates + atom one-hot features) where embeddings must be provably invariant to atom ordering, translation, and rotation — e.g. as an encoder for MS/MS, RT, or CCS prediction.

## When NOT to use

- The molecule has no 3D conformer (input is only a 2D graph or SMILES) — MolConv's geometry channels require xyz coordinates, so generate ETKDGv3 conformers first via molecules_to_records/sdf2pkl
- The task is chiral/enantiomer discrimination with a purely E(3)-invariant encoder — an E(3) encoder provably cannot distinguish enantiomers, so the SE(3) chirality mode is required
- Geometry is computed from absolute atom positions (the deprecated MolConv1 formulation) — this breaks translation invariance and was removed in v1.4.0

## Inputs

- 3D molecular point set as [max_atom_num, 21] array (3 xyz coordinates + 18 atom one-hot features)
- Covalent-style fixed neighbor graph from preprocessing (data_utils.bond_graph_array) with self-pointing padding for unused atom slots
- Layer weights (randomly initialized is sufficient for invariance testing; no checkpoint required)

## Outputs

- Per-atom point representations x_i updated by neighbor aggregation (distance-filter W1 and direction-filter W2 channels)
- Molecular embedding features, e.g. from molnet_engine.save_features() returning (ids, features)
- molconv_invariance_report.json with per-transformation maximum absolute deviation values and pass/fail flags

## How to apply

Instantiate a MolConv layer from molnetpack.molconv with randomly initialized weights (no trained checkpoint is needed since invariance is architectural). Build the input as a [max_atom_num, 21] array (3 xyz coordinates + 18 atom one-hot features) with the covalent-style fixed neighbor graph supplied by preprocessing (data_utils.bond_graph_array), using self-pointing padding for unused slots. At each layer, update each point representation xi by summing over its neighbors: the pairwise distance applies the distance filter W1, the inner-product-based angle term applies the direction filter W2, and both are multiplied element-wise with the neighbor feature xj (Eq. 3). Critically, recompute geometry from relative displacements ⟨xj−xi, xl−xi⟩ after centering on the real-atom centroid, never from absolute coordinates — an earlier variant (MolConv1) computed geometry from absolute atom positions and was removed in v1.4.0 for failing translation invariance. Then apply atom-list permutation, random translation, random rotation, and mirror reflection to the input and compare embeddings; expect floating-point-exact agreement in float64 for permutation/translation/rotation, and sign-flip sensitivity under reflection only when the layer runs in SE(3) chirality mode (chirality=True), since a purely E(3) encoder cannot model enantiomer differences.

## Related tools

- **molnetpack** (Provides the MolConv layer implementation (molnetpack.molconv) used to build and test the GNN layer) — https://pypi.org/project/molnetpack/
- **3DMolMS** (The 3DMolMS model whose encoder contains the MolConv layer being implemented; installed via molnetpack or from source) — https://github.com/JosieHong/3DMolMS
- **PyTorch** (Backend for instantiating the MolConv layer and running the transformed inputs (must be installed separately)) — https://pytorch.org/get-started/locally/
- **NumPy** (Generates random point sets and records per-transformation deviation values in float64 for the invariance report)

## Examples

```
pytest tests/test_encoder_invariance.py tests/test_se3_invariance.py -q
```

## Evaluation signals

- Maximum absolute embedding deviation ≈ 0 (floating-point-exact in float64) under atom-list permutation, translation, and rotation, matching the paper's E(3) + permutation invariance claim
- Embeddings differ (sign-flip sensitivity) under mirror reflection when chirality=True (SE(3) mode); identical under reflection only for E(3) mode
- The package's own suites pass: pytest tests/test_encoder_invariance.py tests/test_se3_invariance.py -q
- molconv_invariance_report.json contains all pass/fail flags set to pass with recorded max-deviation values
- Geometry is derived from relative displacements ⟨xj−xi, xl−xi⟩ after centering on the real-atom centroid, not absolute coordinates

## Limitations

- MolConv1 computed geometry from absolute atom positions and was removed in v1.4.0 because it was not translation-invariant — do not reuse that formulation
- An E(3) encoder is provably unable to model enantiomer differences (it must output one value for both), so SE(3) is required for chiral separation
- Input constraints from the package: atom number ≤ 300 and atom types limited to C, O, N, H, P, S, F, Cl, B, Br, I
- Exact invariance holds only in float64; floating-point tolerance and centroid centering on real atoms (excluding padding) must be handled in the test harness

## Evidence

- [methods] the layer describes geometry through relative positions and angles rather than absolute coordinates, so the molecule's pose does not matter.: "the layer describes geometry through relative positions and angles
rather than absolute coordinates, so the molecule's pose does not matter."
- [methods] An earlier version of this layer (MolConv1) computed geometry from absolute atom positions and was removed in v1.4.0 because it was not translation-invariant: "An earlier version of this layer (`MolConv1`) computed geometry from absolute atom
positions and was removed in v1.4.0 because it was not translation-invariant"
- [methods] An E(3) encoder is provably unable to model enantiomer differences (it must output one value for both), so SE(3) is required for chiral separation: "An E(3) encoder is **provably unable** to model enantiomer differences (it must
output one value for both), so SE(3) is *required* for chiral separation"
- [readme] v1.4.0 changes the encoder to aggregate over the covalent bond graph, so its geometry channels carry bond lengths and bond angles: [evidence span withheld — anchor status: external_doc]
