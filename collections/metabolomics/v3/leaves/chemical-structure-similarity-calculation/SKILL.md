---
name: chemical-structure-similarity-calculation
description: Use when building a weighted molecular network for annotation propagation
  and you need edge weights that blend MetFrag candidate scores, spectral cosine similarity
  and structural similarity of candidate compounds.
when_to_use_negative:
- When the molecular network already provides reliable edge weights and no MetFrag
  candidate scores are available, since the weight function requires per-candidate
  MetFrag annotation scores as input
- When annotating isolated spectra with no molecular network, since structural-similarity
  edge weights depend on pairs of connected network nodes
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0154
tools:
- name: ChemWalker
  role: Python library that assembles the weighted graph from MetFrag scores, spectral
    cosine similarity and chemical structural similarity via the MetFusion-style Fusion
    weight function
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: MetFrag
  role: Performs the in silico combinatorial fragmentation that yields the per-candidate
    annotation scores consumed by the edge-weight calculation
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Provides the Fusion scoring concept used to combine MetFrag scores, spectral
    similarity and structural similarity into edge weights
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Source of the spectral network and spectral cosine similarity scores between
    nodes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:9fbd54bc41d98554df0765d3132e6d8a3b4fdcb7c128f45efb27dab06f4fb4fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - GNPS (Global Natural Products Social Molecular Networking)
  - MetFrag
  - MetFusion
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# chemical-structure-similarity-calculation

## Summary

Compute pairwise chemical (structural) similarity among candidate structures produced by MetFrag in silico fragmentation, and combine these similarities with spectral cosine scores and MetFrag annotation scores through a MetFusion-style sigmoid weight function to derive the edge weights w_ij of the ChemWalker weighted graph.

## When to use

Use when building a weighted molecular network for annotation propagation and you need edge weights that blend MetFrag candidate scores, spectral cosine similarity and structural similarity of candidate compounds.

## When NOT to use

- When the molecular network already provides reliable edge weights and no MetFrag candidate scores are available, since the weight function requires per-candidate MetFrag annotation scores as input
- When annotating isolated spectra with no molecular network, since structural-similarity edge weights depend on pairs of connected network nodes

## Inputs

- MetFrag CL in silico fragmentation results with per-candidate annotation scores for each node
- Spectral cosine similarity scores between molecular-network nodes (from spectral library matching, e.g. GNPS)
- Candidate chemical structures (SMILES/InChI) for each node from MetFrag database searches
- Alpha weighting parameter of the MetFusion-style Fusion score

## Outputs

- Weighted graph in graphml format whose edge weights w_ij encode combined MetFrag-score averages, spectral cosine similarity and chemical structural similarity
- Pairwise chemical (structural) similarity matrix among candidate structures

## How to apply

Parse the MetFrag CL results to obtain per-candidate annotation scores for each node, compute the spectral cosine similarity between connected nodes of the molecular network (from GNPS-style spectral library matching), and compute pairwise structural similarity between the candidate structures of the connected nodes. Combine the alpha-weighted average of the two nodes' MetFrag scores (weight alpha) with the sigmoid of the spectral cosine score multiplied by the chemical similarity of the candidate structures (weight 1-alpha), following the MetFusion Fusion scoring concept, to produce edge weights w_ij. Emit the node set and these edge weights as a graphml file that the random walk consumes. The alpha parameter is the key decision point: it shifts the balance between fragmenter confidence (MetFrag scores) and structural/spectral evidence on the edges. Validation is end-to-end: the graph must be readable, finite, and have non-negative weights on every edge, since the paper evaluates this mechanism only through the downstream random-walk propagation rather than a standalone quantitative criterion.

## Related tools

- **ChemWalker** (Python library that assembles the weighted graph from MetFrag scores, spectral cosine similarity and chemical structural similarity via the MetFusion-style Fusion weight function) — https://github.com/computational-chemical-biology/ChemWalker
- **MetFrag** (Performs the in silico combinatorial fragmentation that yields the per-candidate annotation scores consumed by the edge-weight calculation) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Provides the Fusion scoring concept used to combine MetFrag scores, spectral similarity and structural similarity into edge weights)
- **GNPS (Global Natural Products Social Molecular Networking)** (Source of the spectral network and spectral cosine similarity scores between nodes)

## Examples

```
pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- The output graphml file loads without errors and contains finite, non-negative edge weights on every edge
- Every edge weight reflects the MetFusion-style combination: an alpha-weighted average of MetFrag scores plus a (1-alpha)-weighted sigmoid(spectral cosine * chemical similarity) term
- Node set of the graph matches the MetFrag candidate sets and the molecular-network nodes being annotated
- Downstream random walk over the graph propagates scores from spectral-library-match seed nodes to nodes not directly connected to a spectral match

## Limitations

- The edge-weight mechanism is evaluated only as part of the end-to-end random-walk propagation, not through a standalone quantitative criterion
- Structural similarity depends on the quality of the MetFrag candidate structures, so errors in in silico fragmentation propagate into edge weights
- NAP-style propagation only reaches first neighbors of spectral matches; the value of this weighting is realized only when combined with random-walk propagation that reaches more distant nodes

## Evidence

- [results] Edge weight formulation combining MetFrag scores, spectral cosine and chemical similarity: [evidence span withheld — anchor status: not_found]
- [abstract] Graph construction departing from spectral network plus structural similarity: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function, producing a"
- [methods] Validation of weighted graph readability for the random walk: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [methods] MetFrag provides the per-candidate annotation scores: "ChemWalker uses MetFrag for in silico annotation"
- [readme] ChemWalker is a python package using random walk on candidate structures: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]"
