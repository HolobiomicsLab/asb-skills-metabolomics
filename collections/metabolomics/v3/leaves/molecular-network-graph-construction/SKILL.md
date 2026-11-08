---
name: molecular-network-graph-construction
description: Use when a molecular network's spectral library matches need candidate-structure-aware
  edge weights before random-walk annotation propagation, and MetFrag candidate scores
  plus cosine similarities are available.
when_to_use_negative:
- When the goal is only local propagation to first neighbors of spectral matches —
  NAP-style propagation already covers that, and ChemWalker's weighted-graph random
  walk is designed to reach nodes not directly connected to spectral library matches.
- When MetFrag in silico fragmentation results or candidate structures are unavailable,
  since the edge weights cannot be computed without per-candidate annotation scores.
- When the input is already a valid weighted graph in graphml format with final edge
  weights — this skill is for constructing that graph, not for re-weighting it.
edam_operation: http://edamontology.org/operation_3755
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ChemWalker
  role: Python package that assembles the MetFusion-weighted graph and drives the
    downstream random walk over it
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: MetFrag
  role: Performs combinatorial in silico fragmentation of candidate structures to
    supply per-candidate annotation scores used in the edge-weight computation
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Supplies the Fusion scoring/weight-function concept that combines spectral
    library ranking with structural similarity into edge weights
- name: GNPS
  role: Source of the molecular network and spectral cosine similarity scores used
    as one component of the edge-weight formula
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:36ccc93001d2b52232a52f514f61d4d13cb41ea1c1bbfdddc3c6276bf9ecddf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - GNPS
  - MetFrag
  - MetFusion
  edam_operation: http://edamontology.org/operation_3755
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# molecular-network-graph-construction

## Summary

Constructs a weighted graph for annotation propagation by fusing spectral-network cosine similarities with structural similarity of in silico fragmentation candidates via the MetFusion scoring function, yielding a graphml file consumed by the ChemWalker random walk.

## When to use

Use when a molecular network's spectral library matches need candidate-structure-aware edge weights before random-walk annotation propagation, and MetFrag candidate scores plus cosine similarities are available.

## When NOT to use

- When the goal is only local propagation to first neighbors of spectral matches — NAP-style propagation already covers that, and ChemWalker's weighted-graph random walk is designed to reach nodes not directly connected to spectral library matches.
- When MetFrag in silico fragmentation results or candidate structures are unavailable, since the edge weights cannot be computed without per-candidate annotation scores.
- When the input is already a valid weighted graph in graphml format with final edge weights — this skill is for constructing that graph, not for re-weighting it.

## Inputs

- MetFrag in silico fragmentation results (per-candidate annotation scores)
- Molecular network with spectral cosine similarity scores between connected nodes (e.g. from GNPS spectral library matching)
- Candidate structures for each node (for chemical/structural similarity computation)
- Alpha weighting parameter for the MetFusion-style score combination

## Outputs

- Weighted graph in graphml format with finite, non-negative edge weights on all edges
- Edge weights w_ij combining MetFrag scores, spectral cosine similarity, and structural similarity

## How to apply

Parse the MetFrag in silico fragmentation results to obtain per-candidate annotation scores, compute spectral cosine similarity between connected molecular-network nodes from spectral library matching, and compute chemical (structural) similarity among candidate structures. Combine the two nodes' MetFrag scores as an alpha-weighted average with the sigmoid of the spectral cosine score multiplied by the structural similarity, following the MetFusion Fusion weighting (edge weight w_ij, weight 1-alpha for the averaged MetFrag term). Assemble nodes and edge weights into a graphml-format weighted graph that the ChemWalker random walk consumes, with seed nodes corresponding to spectral library matches.

## Related tools

- **ChemWalker** (Python package that assembles the MetFusion-weighted graph and drives the downstream random walk over it) — https://github.com/computational-chemical-biology/ChemWalker
- **MetFrag** (Performs combinatorial in silico fragmentation of candidate structures to supply per-candidate annotation scores used in the edge-weight computation) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Supplies the Fusion scoring/weight-function concept that combines spectral library ranking with structural similarity into edge weights)
- **GNPS** (Source of the molecular network and spectral cosine similarity scores used as one component of the edge-weight formula)

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- The resulting weighted graph is a valid, readable graphml file with finite, non-negative edge weights on all edges
- Edge weights reproduce the MetFusion-style formulation: alpha-weighted average of the two nodes' MetFrag scores combined (weight 1-alpha) with the sigmoid of spectral cosine score multiplied by chemical similarity
- Seed nodes in the graph correspond to spectral library matches, matching the article's design where random walk departs from seed nodes
- Every connected edge carries a weight derived from both spectral cosine similarity and structural similarity of candidate structures (no zero-weight edges from missing components)

## Limitations

- ChemWalker was tested on MetFrag2.3-CL.jar; other MetFrag builds may not be compatible with the parsing of candidate scores.
- The paper evaluates this mechanism only as part of the end-to-end random-walk propagation, not through a standalone quantitative criterion, so edge weights cannot be validated in isolation.
- The weighted-graph construction requires non-negative, positive weights, since the downstream random walk (pyrwr) is defined on positively weighted networks.
- Propagation depends on the spectral library matches being present as seed nodes; nodes with no spectral match and no structural similarity path receive no information.

## Evidence

- [abstract] weighted-graph construction combines MetFrag scores, spectral cosine and structural similarity via MetFusion: "we have used MetFusion Scoring function to create a weight function, producing a weighted graph"
- [abstract] Random walk departs from seed nodes represented by spectral library matches: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [readme] ChemWalker was tested on MetFrag2.3.-CL.jar: "ChemWalker was tested on [MetFrag2.3.-CL.jar](http://ccbl.fcfrp.usp.br/ccbl/MetFrag2.3-CL.jar), download MetFrag CL [here](https://ipb-halle.github.io/MetFrag/projects/metfragcl/)"
- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation"
- [readme] pyrwr RWR only positive weights allowed: [evidence span withheld — anchor status: external_doc]
