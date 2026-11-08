---
name: molecular-graph-breadth-first-traversal
description: Use when building MS-relevant topological feature vectors from rooted
  fragment graphs — i.e., you need a canonical, index-invariant ordering of fragment
  atoms (nearest to the cleaved-bond root) before constructing an adjacency-tensor
  representation.
when_to_use_negative:
- The input is already a flat feature table or fingerprint (e.g. CSI:FingerID-style
  1415-bit fingerprint) rather than a rooted fragment graph
- You need whole-molecule descriptors rather than fragment-local topology relative
  to a cleaved-bond root — the N=10 nearest-vertex selection deliberately restricts
  context to atoms within ~2 bonds of the root
- Bonds or atom types fall outside the encoded vocabularies (single/double/triple/higher/aromatic/conjugated;
  C, N, O, P, S, other), since these map to zero vectors or the 'other' bucket
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: CFM-ID 4.0
  role: 'Origin and consumer of the traversal: the two-pass BFS canonical indexing
    feeds the adjacency-matrix feature extractor whose output trains CFM-ID 4.0''s
    fragmentation transition probabilities'
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: RDKit
  role: Used (with Python) to parse fragment structures and traverse the molecular
    graph (vertices, edges, bond orders) when re-implementing the heuristic-label
    BFS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:c2520488d225239ee459c71b2db3c078d2e648dca34a50e55ed2b11a8ca5fde0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# molecular-graph-breadth-first-traversal

## Summary

Canonical two-pass breadth-first traversal of a rooted fragment graph that assigns root-distance-based vertex indices, enabling CFM-ID 4.0's adjacency-matrix connectivity-matrix feature representation for learning ESI-MS/MS fragmentation transition probabilities.

## When to use

Use when building MS-relevant topological feature vectors from rooted fragment graphs — i.e., you need a canonical, index-invariant ordering of fragment atoms (nearest to the cleaved-bond root) before constructing an adjacency-tensor representation.

## When NOT to use

- The input is already a flat feature table or fingerprint (e.g. CSI:FingerID-style 1415-bit fingerprint) rather than a rooted fragment graph
- You need whole-molecule descriptors rather than fragment-local topology relative to a cleaved-bond root — the N=10 nearest-vertex selection deliberately restricts context to atoms within ~2 bonds of the root
- Bonds or atom types fall outside the encoded vocabularies (single/double/triple/higher/aromatic/conjugated; C, N, O, P, S, other), since these map to zero vectors or the 'other' bucket

## Inputs

- Rooted fragment graph G = (V, E, VL, EL, R) with atom-type vertex labels and bond-type edge labels, root set to one of the two atoms of the cleaved bond
- Fragment structures exported from the CFM-ID fragmentation graph (e.g. as MOL/SDF block parseable by RDKit)

## Outputs

- Canonical vertex index assignment reflecting topological distance from the root
- Nv x Nv x Dv bond tensor Tadj (one-hot bond type) and Ne x De atom tensor Tvertex (atom type + neighborhood atom count)
- Flattened 1-dimensional concatenated connectivity-matrix feature vector ready for the CFM-ID training algorithm

## How to apply

Represent each fragment as a rooted graph G = (V, E, VL, EL, R), with the root vertex set to one of the two atoms previously connected to the cleaved bond. Run a first breadth-first traversal assigning each vertex a 3-character bond-order-atom-type heuristic string (e.g. '1 C', '2Cl') built from its own pair plus alphanumerically sorted bracketed child labels (e.g. '1 C[1 O][2 C]'); when multiple edges reach the same vertex, keep the edge on the shortest path to the root, breaking ties by lowest bond order. Run a second BFS assigning vertex indices, visiting children in alphanumeric order of their heuristic labels, so larger indices correspond to atoms farther from the root. Select the nearest N=10 vertices to the root (threshold chosen to cover all atoms within 2 bonds), build the subgraph adjacency matrix Madj, encode bonds as one-hot vectors over {single, double, triple, higher orders, aromatic, conjugated} and atoms over type {carbon, nitrogen, oxygen, phosphorus, sulfur, other} plus neighborhood atom count {1–4, 5 or more}, then flatten the un-duplicated (i<j) part of Tadj with all of Tvertex into one 1-D concatenated feature vector. The rationale (per CFM-ID 4.0) is that this representation encodes MS-relevant topological features of fragments so transition probabilities can be learned from molecular bond topology rather than hand-coded rules alone.

## Related tools

- **CFM-ID 4.0** (Origin and consumer of the traversal: the two-pass BFS canonical indexing feeds the adjacency-matrix feature extractor whose output trains CFM-ID 4.0's fragmentation transition probabilities) — https://bitbucket.org/wishartlab/cfm-id-code/
- **RDKit** (Used (with Python) to parse fragment structures and traverse the molecular graph (vertices, edges, bond orders) when re-implementing the heuristic-label BFS)

## Examples

```
from rdkit import Chem; frag = Chem.MolFromMolBlock(rooted_fragment_molblock); labels = {a.GetIdx(): ''.join(sorted(f'{b.GetBondType()} {a.GetSymbol()}' for b in a.GetBonds())) for a in frag.GetAtoms()}
```

## Evaluation signals

- Canonicalization invariant: two input graphs with the same underlying structure and same root atom but different vertex indexing must produce identical flattened feature vectors
- Each vertex's heuristic label is exactly 3 characters of bond-order + atom type plus alphanumerically sorted bracketed child labels (e.g. '1 C[1 O][2 C]')
- In the second BFS, index numbers increase monotonically with topological distance from the root (children visited in alphanumeric order of heuristic labels)
- The selected subgraph covers all atoms within 2 bonds of the root (N=10 threshold must be large enough for this)
- The final vector length is fixed: un-duplicated (i<j) entries of Tadj plus all of Tvertex, concatenated in that order

## Limitations

- Feature window is limited to the N=10 nearest vertices to the root; topological context beyond ~2 bonds from the cleaved bond is discarded
- Atom and bond vocabularies are coarse (atom types collapsed to C, N, O, P, S, 'other'; neighborhood counts capped at '5 or more'), so stereochemical and some heteroatom detail is lost
- Deeper fragmentation graphs introduce extra computational complexity, and the paper notes room for improvement in both accuracy and runtime

## Evidence
