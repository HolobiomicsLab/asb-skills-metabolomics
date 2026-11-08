---
name: weighted-graph-probability-computation
description: Use when you have a weighted graph combining a spectral network and structural-similarity
  edges among MetFrag candidate structures, seed nodes from spectral library matches,
  and need per-node walking probabilities for annotation propagation.
when_to_use_negative:
- The graph is not positively weighted — pyrwr's RWR is defined only on positively
  weighted networks, so negative or zero edge weights are invalid input
- No spectral library match seeds are available for the component — the walk departs
  from seed nodes represented by spectral library matches
- The goal is in silico candidate generation/fragmentation itself — that is performed
  upstream by MetFrag, not by the random-walk step
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: ChemWalker
  role: Python package that orchestrates the random-walk-based propagation of spectral
    library match identities through candidate structures on the weighted graph
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Provides the random-walk-with-restart computation (power iteration with numpy/scipy,
    optional GPU via pytorch) used to score candidate-structure nodes from the seed
    nodes
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFusion
  role: Its scoring function creates the weight function producing the weighted graph
    that the random walk operates on
- name: MetFrag
  role: Performs the in silico combinatorial fragmentation that supplies the candidate
    structures among which similarity edges are weighted
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:35ca1114c2b57356702011151c3671f50a27cfec891a59d471d00b09cc623f30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - pyrwr
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# weighted-graph-probability-computation

## Summary

Computes walking probabilities for candidate-structure nodes on a weighted molecular-network graph by running a random walk with restart (via pyrwr) from seed nodes corresponding to spectral library matches, as implemented in ChemWalker. This extends annotation propagation beyond the first neighbors of spectral matches, which is the limitation of NAP.

## When to use

Use when you have a weighted graph combining a spectral network and structural-similarity edges among MetFrag candidate structures, seed nodes from spectral library matches, and need per-node walking probabilities for annotation propagation.

## When NOT to use

- The graph is not positively weighted — pyrwr's RWR is defined only on positively weighted networks, so negative or zero edge weights are invalid input
- No spectral library match seeds are available for the component — the walk departs from seed nodes represented by spectral library matches
- The goal is in silico candidate generation/fragmentation itself — that is performed upstream by MetFrag, not by the random-walk step

## Inputs

- Weighted graph combining the spectral network and structural-similarity edges among MetFrag candidate structures, scored by the MetFusion weight function (adjacency/edge-list table loadable with pandas)
- Set of seed node identifiers corresponding to spectral library matches (at most 10% of all nodes)

## Outputs

- Table of walking probabilities for every candidate-structure node, keyed by node identifier, aggregated across connected components (probabilities.csv)

## How to apply

Load the weighted graph (built upstream via the MetFusion scoring function from the spectral network plus structural similarity among MetFrag candidates) as an adjacency/edge-list table with pandas. Mark spectral library match nodes as seeds, enforcing that seeds constitute at most 10% of all nodes. Split the graph into connected components (molecular families) and run the random-walk-with-restart routine from pyrwr on each component, seeded at the spectral-match nodes; pyrwr requires positively weighted networks and non-negative integer node ids. Collect the walking probability of every candidate-structure node per component into a single table keyed by node identifier and write it to probabilities.csv.

## Related tools

- **ChemWalker** (Python package that orchestrates the random-walk-based propagation of spectral library match identities through candidate structures on the weighted graph) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Provides the random-walk-with-restart computation (power iteration with numpy/scipy, optional GPU via pytorch) used to score candidate-structure nodes from the seed nodes) — https://github.com/jinhongjung/pyrwr
- **MetFusion** (Its scoring function creates the weight function producing the weighted graph that the random walk operates on)
- **MetFrag** (Performs the in silico combinatorial fragmentation that supplies the candidate structures among which similarity edges are weighted) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
from pyrwr.ppr import PPR; ppr = PPR(); ppr.read_graph(input_graph, 'undirected'); r = ppr.compute(seeds, c=0.15, epsilon=1e-9, max_iters=100)
```

## Evaluation signals

- Seeds constitute at most 10% of all nodes before running the walk
- The graph was first split into connected components and the random walk was run per component (per molecular family), not on the full graph at once
- Output probabilities.csv contains a walking probability for every candidate-structure node in each component, keyed by node identifier
- All edge weights are positive and node ids are non-negative integers, matching pyrwr's RWR input requirements
- Probabilities allow information propagation to reach nodes not directly connected to the spectral library match (unlike NAP's first-neighbor propagation)

## Limitations

- RWR in pyrwr is defined on positively weighted networks; only positive weights are allowed, and redundant edge weights are summed
- pyrwr RWR accepts only a single seed per query; multiple seeds require the PPR query type
- Node ids in the pyrwr input edge list must be non-negative integers
- ChemWalker was tested on MetFrag2.3-CL.jar; upstream MetFrag in silico annotation is a prerequisite

## Evidence

- [abstract] Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function, producing a: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function, producing a"
- [abstract] This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches): "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [abstract] One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match: "One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match"
- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]"
- [readme] RWR is defined on positively weighted networks; thus, only positive weights are allowed.: [evidence span withheld — anchor status: external_doc]
- [methods] conda env create -f environment.yml
conda activate chemwalker: "conda env create -f environment.yml
conda activate chemwalker"
