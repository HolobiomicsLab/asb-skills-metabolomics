---
name: adjacency-matrix-tensor-construction
description: Use when converting rooted molecular fragment graphs (atom/bond-labeled,
  rooted at a cleaved-bond atom) into fixed-length topological feature vectors for
  training or applying CFM-ID-style fragmentation transition models.
when_to_use_negative:
- The input is not a rooted fragment graph — e.g. you need whole-molecule descriptors,
  fingerprints, or spectral features rather than topological features of cleavage-bond-rooted
  fragments
- Fragments whose relevant atoms extend beyond 2 bonds from the root and beyond the
  N=10 nearest-vertex selection window, where the truncated subgraph loses structurally
  important atoms
- You need rule-based rather than learned fragmentation prediction — CFM-ID 4.0's
  expanded hand-written rule predictor (acylcarnitines, acylcholines, flavonols, flavones,
  flavanones, flavonoid glycosides) does not use this learned feature representation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CFM-ID 4.0
  role: Consumer of the flattened connectivity-matrix feature vector for learning
    fragmentation transition probabilities; also available as web server and Docker
    image
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: RDKit
  role: Python cheminformatics library used to parse structures and extract atom-type
    and bond-type labels for building the fragment graphs and tensors
- name: Python
  role: Implementation language for the BFS traversals, one-hot tensor construction,
    and flattening step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:b6aa0d2a43e36490b4fdc026830098e2dde6a1fe4bd2d7d3b79b95f0842642f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# adjacency-matrix-tensor-construction

## Summary

Builds the connectivity-matrix (adjacency-tensor) chemical structure representation used by CFM-ID 4.0, encoding MS-relevant topological features of rooted fragment graphs as a flattened 1-D vector for learning fragmentation transition probabilities.

## When to use

Use when converting rooted molecular fragment graphs (atom/bond-labeled, rooted at a cleaved-bond atom) into fixed-length topological feature vectors for training or applying CFM-ID-style fragmentation transition models.

## When NOT to use

- The input is not a rooted fragment graph — e.g. you need whole-molecule descriptors, fingerprints, or spectral features rather than topological features of cleavage-bond-rooted fragments
- Fragments whose relevant atoms extend beyond 2 bonds from the root and beyond the N=10 nearest-vertex selection window, where the truncated subgraph loses structurally important atoms
- You need rule-based rather than learned fragmentation prediction — CFM-ID 4.0's expanded hand-written rule predictor (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides) does not use this learned feature representation

## Inputs

- Rooted fragment graph G = (V, E, VL, EL, R) with atom-type vertex labels, bond-type edge labels, and root vertex set to one of the two atoms previously connected to the cleaved bond
- Molecule or fragment structure parseable with RDKit (Python) or CFM-ID 4.0 input

## Outputs

- Nv x Nv x Dv adjacency tensor Tadj with one-hot encoded bond types {single, double, triple, higher orders, aromatic, conjugated}
- Ne x De vertex tensor Tvertex encoding atom type and neighborhood atom count
- Flattened 1-D connectivity-matrix feature vector (un-duplicated i<j part of Tadj concatenated with all of Tvertex) suitable for CFM-ID training

## How to apply

Represent each fragment as a rooted graph G = (V, E, VL, EL, R) with atom-type vertex labels and bond-type edge labels, rooting at one of the two atoms formerly joined by the cleaved bond. Run two breadth-first traversals: the first assigns each vertex a 3-character bond-order-atom-type heuristic string (e.g. '1 C', '2Cl', or '1 C[1 O][2 C]' with alphanumerically sorted bracketed child labels, choosing shortest-path-to-root edges and breaking ties by lowest bond order); the second assigns canonical indices, visiting children in alphanumeric order of heuristic labels so higher indices mean greater topological distance from the root. Select the nearest N=10 vertices (a threshold large enough to cover all atoms within 2 bonds of the root) and build the Nv x Nv x Dv tensor Tadj whose (i,j) entry is a one-hot bond type from {single, double, triple, higher orders, aromatic, conjugated} when an edge exists, and the Ne x De tensor Tvertex encoding atom type from {carbon, nitrogen, oxygen, phosphorus, sulfur, other} and neighborhood atom count from {1, 2, 3, 4, 5 or more}. Flatten the un-duplicated (i<j) part of Tadj together with all of Tvertex into the 1-D concatenated vector fed to the CFM-ID training algorithm.

## Related tools

- **CFM-ID 4.0** (Consumer of the flattened connectivity-matrix feature vector for learning fragmentation transition probabilities; also available as web server and Docker image) — https://bitbucket.org/wishartlab/cfm-id-code/
- **RDKit** (Python cheminformatics library used to parse structures and extract atom-type and bond-type labels for building the fragment graphs and tensors)
- **Python** (Implementation language for the BFS traversals, one-hot tensor construction, and flattening step)

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(frag_smiles); root = mol.GetAtomWithIdx(root_idx)  # build rooted graph, then run 2 BFS passes to fill Tadj (Nv x Nv x Dv) and Tvertex (Ne x De) and flatten i<j part + Tvertex
```

## Evaluation signals

- Canonicalization invariant: two input graphs with the same underlying structure and the same root atom but different vertex indexing must produce identical flattened feature vectors
- Shape check: Tadj has dimensions Nv x Nv x Dv and Tvertex has dimensions Ne x De, with N=10 nearest vertices selected by BFS index
- Bond-type check: every non-zero (i,j) vector in Tadj is a valid one-hot encoding from {single, double, triple, higher orders, aromatic, conjugated}; entries for absent edges are zero vectors
- Vertex-feature check: each Tvertex entry encodes one of {carbon, nitrogen, oxygen, phosphorus, sulfur, other} and a neighborhood atom count from {1, 2, 3, 4, 5 or more}
- Rooting check: the root vertex is one of the two atoms previously connected to the cleaved bond, and BFS indices increase with topological distance from the root

## Limitations

- The N=10 nearest-vertex selection truncates the fragment representation; atoms beyond the threshold (though it covers all atoms within 2 bonds) are excluded from Tadj and Tvertex
- Feature encoding is limited to six atom types and six bond types, so exotic elements and unusual bond orders are lumped into 'other' / 'higher orders' categories
- Deeper fragmentation graphs introduce extra computational complexity, and CFM-ID 4.0 authors note remaining room for improvement in accuracy and runtime

## Evidence

- [intro] Objective: develop an adjacency matrix based chemical structure representation for CFM-ID 4.0 that extracts topological features of fragments relevant to MS.: "we developed an adjacency matrix based chemical structure representation for CFM-ID 4.0 that extracts topological features of fragments that are MS relevant"
- [intro] CFM-ID 4.0 learns parameters from chemical bond topology.: "we developed an improved version of CFM-ID (version 4.0) that learns its parameters from a detailed representation of the molecule’s chemical bond topology"
- [abstract] Topology-based features improve ESI-MS/MS prediction accuracy.: "This work improves CFM-ID's ability to predict ESI-MS/MS spectra from compounds by (1) learning parameters from features based on the molecular topology"
- [intro] CFM-ID builds a fragmentation graph of transitions before probability estimation.: "CFM-ID first generates all its theoretically possible fragments combinatorially, structured as a fragmentation graph"
