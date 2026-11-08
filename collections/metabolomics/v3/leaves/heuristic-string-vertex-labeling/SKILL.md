---
name: heuristic-string-vertex-labeling
description: Use when building MS-relevant topological features from fragment graphs
  where vertex ordering from a structure reader is arbitrary — i.e., when identical
  rooted subgraphs must map to identical feature vectors regardless of input atom
  indexing.
when_to_use_negative:
- The fragment representation already comes from a canonical, root-aware indexing
  scheme — re-labeling is redundant and may not match CFM-ID's tie-breaking (lowest
  bond order, shortest path to root)
- Fragments whose relevant neighborhood exceeds 2 bonds from the root — the N=10 selection
  threshold is only guaranteed to cover all atoms within 2 bonds
- Aim is general molecular fingerprinting or similarity rather than feeding CFM-ID's
  transition-probability training — this labeling is specific to the adjacency-matrix
  representation
tools:
- name: CFM-ID 4.0
  role: Defines the heuristic-string labeling scheme and consumes the resulting canonical
    adjacency-matrix features when learning fragmentation transition probabilities
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: RDKit
  role: Used in Python to parse structures and extract atom/bond types and bond orders
    for constructing the rooted fragment graphs
- name: Python
  role: Implementation language for the two BFS traversals and canonical labeling
    validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:4c61634df89f15c075ca9447601388b03fae89f7e11cc5f66296cd24a8227803
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Python
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# heuristic-string-vertex-labeling

## Summary

Canonical labeling of rooted fragment graphs via two breadth-first traversals that assign each vertex a 3-character bond-order-atom-type heuristic string (e.g. '1 C[1 O][2 C]'), ordering children alphanumerically so vertex indices reflect topological distance from the cleaved-bond root. This is the labeling step feeding CFM-ID 4.0's adjacency-matrix feature representation.

## When to use

Use when building MS-relevant topological features from fragment graphs where vertex ordering from a structure reader is arbitrary — i.e., when identical rooted subgraphs must map to identical feature vectors regardless of input atom indexing.

## When NOT to use

- The fragment representation already comes from a canonical, root-aware indexing scheme — re-labeling is redundant and may not match CFM-ID's tie-breaking (lowest bond order, shortest path to root)
- Fragments whose relevant neighborhood exceeds 2 bonds from the root — the N=10 selection threshold is only guaranteed to cover all atoms within 2 bonds
- Aim is general molecular fingerprinting or similarity rather than feeding CFM-ID's transition-probability training — this labeling is specific to the adjacency-matrix representation

## Inputs

- Fragment graph G = (V, E, VL, EL, R): atom-type vertex labels, bond-type edge labels, and a root vertex (one of the two atoms previously connected to the cleaved bond)
- Bond-type set: single, double, triple, higher orders, aromatic, conjugated
- Atom-type set: carbon, nitrogen, oxygen, phosphorus, sulfur, other

## Outputs

- Canonical vertex indices per fragment (children visited in alphanumeric order of heuristic labels; larger indices = farther from root)
- Per-vertex 3-character bond-order-atom-type heuristic strings (e.g. '1 C', '2Cl', '1 C[1 O][2 C]')
- Adjacency matrix Madj over the nearest N=10 root vertices, ready for tensor construction (Tadj, Tvertex) and flattening into the 1-dimensional feature vector

## How to apply

Represent each fragment as a rooted graph G = (V, E, VL, EL, R), with the root vertex set to one of the two atoms formerly joined by the cleaved bond. Run a first BFS to assign each vertex a 3-character bond-order-atom-type heuristic string built from its own bond-order-atom-type pair plus alphanumerically sorted bracketed child labels (e.g. '1 C[1 O][2 C]'); when multiple edges lead to the same vertex, choose the edge on the shortest path to the root, breaking ties by lowest bond order. Run a second BFS visiting children in alphanumeric order of their heuristic labels, assigning indices so larger index numbers correspond to atoms farther from the root. Select the nearest N=10 vertices to the root and build the adjacency matrix from that subgraph. Always validate canonicity: two graphs with identical structure and root but different input indexing must produce identical flattened vectors.

## Related tools

- **CFM-ID 4.0** (Defines the heuristic-string labeling scheme and consumes the resulting canonical adjacency-matrix features when learning fragmentation transition probabilities) — https://bitbucket.org/wishartlab/cfm-id-code/
- **RDKit** (Used in Python to parse structures and extract atom/bond types and bond orders for constructing the rooted fragment graphs)
- **Python** (Implementation language for the two BFS traversals and canonical labeling validation)

## Evaluation signals

- Canonicality invariant: two input graphs with the same underlying structure and the same root atom but different vertex indexing must produce identical flattened feature vectors
- Index-distance monotonicity: larger assigned vertex index numbers correspond to atoms farther from the root in the BFS order
- Heuristic-string round-trip: every vertex label is a 3-character bond-order-atom-type string plus alphanumerically sorted bracketed child labels (e.g. '1 C[1 O][2 C]')
- Subgraph coverage: for fragments with ≤2-bond neighborhoods, all such atoms are included in the N=10 selection
- Downstream acceptance: the flattened vector concatenates only the un-duplicated (i<j) part of Tadj plus all of Tvertex, with zero vectors at (i,j) positions lacking an edge

## Limitations

- Canonical ordering depends on alphanumeric tie-breaking; atoms with identical heuristic labels may yield orderings sensitive to the shortest-path / lowest-bond-order edge-choice rule
- The N=10 nearest-vertex cutoff is calibrated for neighborhoods within 2 bonds of the root; larger fragments are truncated, losing long-range topology
- Atom and bond vocabularies are fixed (six atom types, seven bond-type categories incl. 'other'), so exotic elements/orders collapse into coarse bins

## Evidence

- [methods] Run the first breadth-first traversal to assign each vertex a 3-character bond-order-atom-type heuristic string (e.g. '1 C' for a single bond to carbon, '2Cl' for a double bond to chlorine): [evidence span withheld — anchor status: not_found]
- [methods] Run the second breadth-first traversal to assign indices to vertices, visiting child vertices in alphanumeric order of their heuristic labels (smaller labels visited first): [evidence span withheld — anchor status: not_found]
- [methods] Select the nearest N=10 vertices to the root based on their assigned index (threshold large enough to cover all atoms within 2 bonds) and build an adjacency matrix: [evidence span withheld — anchor status: not_found]
