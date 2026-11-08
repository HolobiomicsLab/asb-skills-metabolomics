---
name: rooted-graph-invariance-testing
description: Use when implementing or auditing the adjacency-matrix connectivity feature
  representation of rooted fragment graphs (root = atom adjacent to a cleaved bond),
  where the flattened feature vector must be invariant to arbitrary vertex ordering.
when_to_use_negative:
- The input is already a flat feature table or numeric descriptor matrix — this skill
  operates on rooted fragment graphs prior to flattening, not on downstream ML features
- You need an unrooted whole-molecule representation or features independent of a
  cleavage site — the canonical indexing is defined relative to the root atom (adjacent
  to the cleaved bond)
- You are predicting spectra purely via hand-written fragmentation rules (e.g. MSRB
  class rules) without the machine-learned C2MS component that consumes these topology
  features
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: CFM-ID 4.0
  role: Defines the connectivity-matrix feature representation and the rooted-graph
    BFS canonical indexing that this invariance test validates
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: RDKit
  role: Parses chemical structures/fragments into atom- and bond-labeled graphs used
    to build the rooted graphs and bond/atom tensors
- name: Python
  role: Implements the tensor construction, flattening, and element-wise equality
    assertion of the invariance check
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:977919bad50085ce3ad4f0e17001c506277924ef88dc1a1d05a0c062fa1653a0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rooted-graph-invariance-testing

## Summary

A validation technique for CFM-ID 4.0's connectivity-matrix feature extractor, verifying that the flattened bond-tensor/vertex-tensor vector for a rooted fragment graph is canonical — i.e., identical for two graphs with the same underlying structure and root atom regardless of their original vertex indexing.

## When to use

Use when implementing or auditing the adjacency-matrix connectivity feature representation of rooted fragment graphs (root = atom adjacent to a cleaved bond), where the flattened feature vector must be invariant to arbitrary vertex ordering.

## When NOT to use

- The input is already a flat feature table or numeric descriptor matrix — this skill operates on rooted fragment graphs prior to flattening, not on downstream ML features
- You need an unrooted whole-molecule representation or features independent of a cleavage site — the canonical indexing is defined relative to the root atom (adjacent to the cleaved bond)
- You are predicting spectra purely via hand-written fragmentation rules (e.g. MSRB class rules) without the machine-learned C2MS component that consumes these topology features

## Inputs

- Rooted fragment graph G = (V, E, VL, EL, R) with atom-type vertex labels and bond-type edge labels, root set to one of the two atoms adjacent to the cleaved bond
- A second, index-permuted copy of the same rooted graph (same structure, same root) for the invariance test
- Optionally, RDKit Mol objects or SMILES of fragments from which rooted graphs are built

## Outputs

- 1-dimensional flattened feature vector per fragment (un-duplicated i<j slice of Tadj concatenated with all of Tvertex)
- Boolean pass/fail of the invariance check (element-wise equality of the two flattened vectors)

## How to apply

Represent each fragment as a rooted graph G = (V, E, VL, EL, R) and run the two breadth-first traversals of the CFM-ID 4.0 feature extractor: the first assigns each vertex a 3-character bond-order-atom-type heuristic string (e.g. '1 C' or '1 C[1 O][2 C]', choosing the shortest-path edge and breaking ties by lowest bond order); the second assigns indices by visiting children in alphanumeric order of those labels so larger indices lie farther from the root. Select the nearest N=10 vertices (threshold sized to cover all atoms within 2 bonds of the root), build the Nv x Nv x Dv bond tensor Tadj with one-hot vectors from {single, double, triple, higher orders, aromatic, conjugated} (zero vector when no edge) and the Ne x De atom tensor Tvertex encoding type from {carbon, nitrogen, oxygen, phosphorus, sulfur, other} and neighborhood atom count from {1, 2, 3, 4, 5 or more}. Flatten the un-duplicated (i<j) part of Tadj with all of Tvertex into a single 1-D concatenated vector. Then perform the invariance test: take two input graphs with identical structure and root atom but different vertex indexing, run the full pipeline on each, and assert the flattened vectors are element-wise identical — this confirms the indexing is canonical and reflects topological distance from the root rather than input ordering. Implement the graph parsing (e.g. from RDKit mol objects) and tensor construction in Python, mirroring the CFM-ID 4.0 source.

## Related tools

- **CFM-ID 4.0** (Defines the connectivity-matrix feature representation and the rooted-graph BFS canonical indexing that this invariance test validates) — https://bitbucket.org/wishartlab/cfm-id-code/
- **RDKit** (Parses chemical structures/fragments into atom- and bond-labeled graphs used to build the rooted graphs and bond/atom tensors)
- **Python** (Implements the tensor construction, flattening, and element-wise equality assertion of the invariance check)

## Examples

```
import numpy as np; v1 = flatten(tensors(G1, root)); v2 = flatten(tensors(permute_indices(G1), root)); assert np.array_equal(v1, v2)
```

## Evaluation signals

- Invariance invariant: two input graphs with the same underlying structure and the same root atom but different vertex indexing produce identical flattened feature vectors (element-wise equality)
- Index monotonicity: larger assigned indices correspond to atoms farther (in topological distance) from the root after the second BFS
- Window check: the N=10 nearest-vertex selection covers all atoms within 2 bonds of the root for typical fragment sizes
- Tensor schema: Tadj entries are one-hot over {single, double, triple, higher orders, aromatic, conjugated} when an edge exists and zero vectors otherwise; Tvertex encodes atom type from {C, N, O, P, S, other} and neighborhood count from {1–5+}
- Flattening rule: only the un-duplicated (i<j) part of Tadj is used, concatenated with all of Tvertex — vector length must match that formula exactly

## Limitations

- Only the nearest N=10 vertices to the root are retained, so the feature vector truncates atoms more than ~2 bonds from the root and discels distant topology
- Atom types are limited to the six buckets {carbon, nitrogen, oxygen, phosphorus, sulfur, other}, collapsing elements outside that list into 'other'
- Deeper fragmentation graphs introduce extra computational complexity, which carries over into feature extraction and testing at scale

## Evidence
