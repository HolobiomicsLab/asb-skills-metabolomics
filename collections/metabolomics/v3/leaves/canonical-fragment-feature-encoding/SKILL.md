---
name: canonical-fragment-feature-encoding
description: Use when a fragment must be converted from a rooted molecular graph (e.g.
  from RDKit) into a fixed, index-invariant topological feature vector for training
  or applying CFM-ID 4.0-style fragmentation transition models.
when_to_use_negative:
- Input is already a numeric feature table or spectral vector — this skill operates
  on rooted fragment graphs, not on precomputed features
- The fragment cannot be rooted on a cleaved-bond atom, since canonical indexing and
  topological distance are defined relative to that root
- A representation needing 3D coordinates or bond-energy descriptors is required —
  this encoding is purely 2D topological (adjacency and atom/bond types)
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: CFM-ID 4.0
  role: Framework whose training algorithm consumes the flattened connectivity-matrix
    feature vector to learn fragmentation transition probabilities
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: RDKit
  role: Parses compound structures into molecular graphs from which rooted fragment
    graphs (V, E, VL, EL, R) are built for encoding
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:8263298117b3e8c8265efea6cd2c55746e510584304acb08ff54ef0151581825
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# canonical-fragment-feature-encoding

## Summary

Encodes a rooted fragment graph as a canonical 1-D feature vector by flattening the un-duplicated (i<j) part of a one-hot bond-type adjacency tensor Tadj together with an atom-type/neighbor-count tensor Tvertex, as used by CFM-ID 4.0 to learn ESI-MS/MS fragmentation transition probabilities.

## When to use

Use when a fragment must be converted from a rooted molecular graph (e.g. from RDKit) into a fixed, index-invariant topological feature vector for training or applying CFM-ID 4.0-style fragmentation transition models.

## When NOT to use

- Input is already a numeric feature table or spectral vector — this skill operates on rooted fragment graphs, not on precomputed features
- The fragment cannot be rooted on a cleaved-bond atom, since canonical indexing and topological distance are defined relative to that root
- A representation needing 3D coordinates or bond-energy descriptors is required — this encoding is purely 2D topological (adjacency and atom/bond types)

## Inputs

- Rooted fragment graph G = (V, E, VL, EL, R) with atom-type vertex labels, bond-type edge labels, and root set to one of the two atoms of the cleaved bond
- Molecule or fragment structure parseable with RDKit (e.g. SMILES) from which the fragment graph is derived
- Identity of the cleaved bond's two atoms, used to set the root vertex

## Outputs

- One 1-dimensional concatenated feature vector per fragment: flattened un-duplicated (i<j) part of Tadj (one-hot bond types over {single, double, triple, higher orders, aromatic, conjugated}) plus all of Tvertex (atom type over {C, N, O, P, S, other} and neighborhood atom count over {1-4, 5+})

## How to apply

Root the fragment graph G=(V,E,VL,EL,R) on one of the two atoms previously connected to the cleaved bond, then run two breadth-first traversals: the first assigns each vertex a 3-character bond-order-atom-type heuristic string (e.g. '1 C[1 O][2 C]') choosing the shortest-path edge to the root when multiple edges reach a vertex, and the second assigns indices visiting children in alphanumeric order of these labels so larger indices mean greater topological distance from the root. Select the nearest N=10 vertices (a threshold chosen to cover all atoms within 2 bonds) and build the subgraph adjacency matrix Madj. Construct the Nv x Nv x Dv tensor Tadj with one-hot bond types from {single, double, triple, higher orders, aromatic, conjugated} (zero vector for absent edges) and the Ne x De tensor Tvertex encoding atom type from {carbon, nitrogen, oxygen, phosphorus, sulfur, other} plus neighborhood atom count from {1,2,3,4,5 or more}. Flatten the un-duplicated (i<j) part of Tadj together with all of Tvertex into a single 1-D vector to feed CFM-ID's training algorithm.

## Related tools

- **CFM-ID 4.0** (Framework whose training algorithm consumes the flattened connectivity-matrix feature vector to learn fragmentation transition probabilities) — https://bitbucket.org/wishartlab/cfm-id-code/
- **RDKit** (Parses compound structures into molecular graphs from which rooted fragment graphs (V, E, VL, EL, R) are built for encoding)

## Evaluation signals

- Canonicality invariant: two input graphs with the same underlying structure and the same root atom but different vertex indexing must produce identical flattened feature vectors
- Schema check: the vector equals the flattened un-duplicated (i<j) part of Tadj concatenated with all of Tvertex — no duplicated upper-triangle entries, no missing atom tensor entries
- Vertex-count check: the selected subgraph contains the nearest N=10 vertices by assigned index, a threshold large enough to cover all atoms within 2 bonds of the root
- Value-domain check: every non-zero (i,j) slice of Tadj is a valid one-hot over {single, double, triple, higher orders, aromatic, conjugated}, and atom encodings use only {C, N, O, P, S, other} with neighbor counts in {1,2,3,4,5+}
- Root check: the root vertex is one of the two atoms previously connected to the cleaved bond

## Limitations

- Only the N=10 nearest vertices are encoded, so atoms beyond that window (beyond ~2 bonds from the root) contribute no signal
- Deeper fragmentation graphs introduce extra computational complexity, which propagates to more fragment encodings (article discussion)
- The representation captures topology only; the article notes that encoding more information about chemical bond energies could further improve ESI-MS/MS fragmentation modeling

## Evidence
