---
name: bond-order-atom-type-pair-parsing
description: Use when you have rooted fragment graphs G = (V, E, VL, EL, R) from cleaved
  bonds and need canonical, index-invariant topological features (bond orders, atom
  types, BFS ordering) for ML spectral prediction.
when_to_use_negative:
- The input is a whole unfragmented molecule with no cleaved bond, so no root vertex
  R can be defined — this representation is defined on rooted fragment graphs only
- The analysis needs 3D conformer or bond-energy features rather than 2D topological
  bond/atom type encoding
- A feature table or descriptor matrix already exists and only downstream ML training
  on CFM-ID is required
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: CFM-ID 4.0
  role: Learning algorithm that consumes the flattened connectivity-matrix feature
    vector to estimate fragmentation transition probabilities
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: RDKit
  role: Python cheminformatics library used to parse molecular graphs, bond orders,
    and atom types before feature construction
- name: Python
  role: Implementation language for the BFS traversals, tensor construction, and flattening
    steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:659ef3b5e9dcf23e0f1737e918fb1786cdd3db305c0b144ae4b59727eadd72cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bond-order-atom-type-pair-parsing

## Summary

Parses rooted fragment graphs into bond-order/atom-type pairs and heuristic labels (e.g. '1 C', '2Cl', '1 C[1 O][2 C]') that feed CFM-ID 4.0's adjacency-matrix feature representation of MS-relevant fragment topology.

## When to use

Use when you have rooted fragment graphs G = (V, E, VL, EL, R) from cleaved bonds and need canonical, index-invariant topological features (bond orders, atom types, BFS ordering) for ML spectral prediction.

## When NOT to use

- The input is a whole unfragmented molecule with no cleaved bond, so no root vertex R can be defined — this representation is defined on rooted fragment graphs only
- The analysis needs 3D conformer or bond-energy features rather than 2D topological bond/atom type encoding
- A feature table or descriptor matrix already exists and only downstream ML training on CFM-ID is required

## Inputs

- Rooted fragment graph G = (V, E, VL, EL, R) with atom-type vertex labels and bond-type edge labels, rooted at an atom of the cleaved bond
- Bond types mapped to {single, double, triple, higher orders, aromatic, conjugated}
- Atom types mapped to {carbon, nitrogen, oxygen, phosphorus, sulfur, other} with per-atom neighborhood counts

## Outputs

- 3-character bond-order-atom-type heuristic strings per vertex (e.g. '1 C', '2Cl', '1 C[1 O][2 C]')
- Canonical vertex index assignment ordered by topological distance from the root
- Nv x Nv x Dv bond tensor Tadj and Ne x De vertex tensor Tvertex, flattened into a 1-D concatenated feature vector for CFM-ID training

## How to apply

Represent each fragment as a rooted graph whose root is one of the two atoms previously joined by the cleaved bond, with atom-type vertex labels and bond-type edge labels. Run a first breadth-first traversal assigning each vertex a 3-character bond-order-atom-type heuristic string (e.g. '1 C' for a single bond to carbon, '2Cl' for a double bond to chlorine), extending labels with alphanumerically sorted bracketed child labels (e.g. '1 C[1 O][2 C]'); when multiple edges reach the same vertex, keep the edge on the shortest path to the root, breaking ties by lowest bond order. Run a second BFS visiting children in alphanumeric order of their heuristic labels so larger indices correspond to atoms farther from the root, then select the nearest N=10 vertices (enough to cover all atoms within 2 bonds). Encode bond types from {single, double, triple, higher orders, aromatic, conjugated} and atom types from {carbon, nitrogen, oxygen, phosphorus, sulfur, other} plus neighborhood atom count {1-5+} as one-hot tensors, flattening the un-duplicated (i<j) part of Tadj with all of Tvertex into one feature vector. The rationale is CFM-ID 4.0's claim that learning parameters from detailed chemical bond topology improves ESI-MS/MS transition probability prediction.

## Related tools

- **CFM-ID 4.0** (Learning algorithm that consumes the flattened connectivity-matrix feature vector to estimate fragmentation transition probabilities) — https://bitbucket.org/wishartlab/cfm-id-code/
- **RDKit** (Python cheminformatics library used to parse molecular graphs, bond orders, and atom types before feature construction)
- **Python** (Implementation language for the BFS traversals, tensor construction, and flattening steps)

## Evaluation signals

- Canonicality invariant: two input graphs with the same underlying structure and the same root atom but different vertex indexing must yield identical flattened feature vectors
- Heuristic label schema check: every vertex label is a 3-character bond-order-atom-type string (e.g. '1 C', '2Cl') optionally followed by alphanumerically sorted bracketed child labels
- BFS ordering check: larger vertex indices correspond to atoms farther from the root, with children visited in alphanumeric order of their heuristic labels
- Coverage check: the N=10 nearest-vertex threshold covers all atoms within 2 bonds of the root
- One-hot validity: Tadj entries come only from {single, double, triple, higher orders, aromatic, conjugated}; Tvertex only from the 6 atom types x 5 neighborhood-count classes; only the i<j part of Tadj is kept (un-duplicated)

## Limitations

- Deeper fragmentation graphs introduce extra computational complexity, and the authors note room for improvement in accuracy and runtime
- The representation is developed for QTOF ESI-MS/MS; CFM-ID 4.0 could not handle spectra from non-QTOF instruments at time of publication
- Feature vocabulary is restricted to predefined bond-type and atom-type classes, so atom types outside {C, N, O, P, S} collapse into 'other' and neighborhood counts are capped at '5 or more'

## Evidence

- [intro] Adjacency-matrix representation rationale: "we developed an adjacency matrix based chemical structure representation for CFM-ID 4.0 that extracts topological features of fragments that are MS relevant"
- [intro] Bond-topology learning objective: "we developed an improved version of CFM-ID (version 4.0) that learns its parameters from a detailed representation of the molecule’s chemical bond topology"
- [other] First BFS heuristic labels (source card workflow): [evidence span withheld — anchor status: not_found]
- [other] Canonicality validation step (source card workflow): [evidence span withheld — anchor status: not_found]
