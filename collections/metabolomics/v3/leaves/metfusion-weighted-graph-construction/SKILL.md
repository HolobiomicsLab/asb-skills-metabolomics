---
name: metfusion-weighted-graph-construction
description: Decision support for building a ChemWalker-style weighted graph when
  you have a spectral network plus in silico fragmentation candidate lists (MetFrag
  results) and need edge weights that fuse spectral and structural similarity before
  random-walk propagation.
when_to_use_negative:
- The goal is NAP-style local propagation only to first neighbors of spectral matches
  — ChemWalker's MetFusion-weighted graph plus random walk targets propagation to
  nodes not directly connected to spectral matches.
- No in silico fragmentation candidate lists are available, so structural similarity
  among candidates cannot be fused with the spectral network via the MetFusion scoring
  function.
- The graph contains zero or negative weights — pyrwr's RWR is defined only on positively
  weighted networks.
tools:
- name: ChemWalker
  role: Python package whose weighting scheme uses the MetFusion scoring function
    to combine spectral network and structural similarity among candidates into the
    weighted graph used for random-walk propagation
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: MetFusion
  role: Provides the scoring/Fusion concept for ranking improvement from spectral
    library that is used to create the weight function of the weighted graph
- name: MetFrag
  role: Produces the in silico combinatorial fragmentation results and candidate structures
    whose structural similarity feeds into the MetFusion-weighted graph
  repo: https://github.com/ipb-halle/MetFragRelaunched/releases
- name: pyrwr
  role: Consumes the resulting positively weighted graph to perform the random walk
    that computes candidate-walking probabilities from seed nodes
  repo: https://github.com/jinhongjung/pyrwr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:c58545fa1240d53ec1056512f88b82b8da642f95a786a131c2e7cef3ff0791b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - pyrwr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metfusion-weighted-graph-construction

## Summary

Construct a weighted graph for annotation propagation by combining a spectral (molecular) network with structural similarity among candidate structures using the MetFusion scoring function as the weight function, as implemented in ChemWalker.

## When to use

Decision support for building a ChemWalker-style weighted graph when you have a spectral network plus in silico fragmentation candidate lists (MetFrag results) and need edge weights that fuse spectral and structural similarity before random-walk propagation.

## When NOT to use

- The goal is NAP-style local propagation only to first neighbors of spectral matches — ChemWalker's MetFusion-weighted graph plus random walk targets propagation to nodes not directly connected to spectral matches.
- No in silico fragmentation candidate lists are available, so structural similarity among candidates cannot be fused with the spectral network via the MetFusion scoring function.
- The graph contains zero or negative weights — pyrwr's RWR is defined only on positively weighted networks.

## Inputs

- Spectral (molecular) network with nodes representing mass signals and edges from spectral similarity
- MetFrag in silico combinatorial fragmentation results (candidate structures per node, searched against biologically relevant databases)
- Structural similarity among candidate structures
- Spectral library matches (seed nodes)

## Outputs

- Weighted graph (adjacency/edge list) whose edge weights are produced by the MetFusion scoring function, suitable as pyrwr input
- Positive edge weights amenable to random walk with restart

## How to apply

First obtain in silico combinatorial fragmentation candidates by running MetFrag against biologically relevant databases for each network node. Then, departing from the combination of the spectral network and the structural similarity among candidate structures, apply the MetFusion scoring function as a weight function to produce a weighted graph that fuses spectral-similarity edges with candidate-structure similarity. Positive edge weights are required because the subsequent random walk (via pyrwr) is defined only on positively weighted networks. The resulting weighted graph is the input on which the random walk computes the probability of 'walking' through a set of candidates, departing from seed nodes represented by spectral library matches. This weighting scheme is what lets annotation information propagate beyond the first neighbors of a spectral match, addressing the local-propagation limitation of NAP.

## Related tools

- **ChemWalker** (Python package whose weighting scheme uses the MetFusion scoring function to combine spectral network and structural similarity among candidates into the weighted graph used for random-walk propagation) — https://github.com/computational-chemical-biology/ChemWalker
- **MetFusion** (Provides the scoring/Fusion concept for ranking improvement from spectral library that is used to create the weight function of the weighted graph)
- **MetFrag** (Produces the in silico combinatorial fragmentation results and candidate structures whose structural similarity feeds into the MetFusion-weighted graph) — https://github.com/ipb-halle/MetFragRelaunched/releases
- **pyrwr** (Consumes the resulting positively weighted graph to perform the random walk that computes candidate-walking probabilities from seed nodes) — https://github.com/jinhongjung/pyrwr

## Examples

```
conda env create -f environment.yml && conda activate chemwalker && pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- All edge weights in the constructed graph are positive — RWR via pyrwr is only valid on positively weighted networks.
- Edge weights must reflect the MetFusion scoring function applied to the combination of spectral network edges and structural similarity among candidate structures.
- The weighted graph, when run through the random walk, should propagate annotation information to nodes not directly connected to spectral library matches (beyond first neighbors).
- Reproducibility check: ChemWalker can be installed in the dedicated conda environment (`conda env create -f environment.yml; pip install git+.../ChemWalker.git`) and re-run to regenerate the weighted graph.

## Limitations

- Weight construction depends on candidate quality from MetFrag in silico combinatorial fragmentation searched against biologically relevant databases.
- The approach's differentiation from NAP on the 555-spectra NIST network could not be statistically established for top-10 candidate classification, so weighting benefits may not be separable from NAP on small networks.
- pyrwr requires strictly positive edge weights; redundant weighted edges are summed, which must be accounted for when combining spectral and structural similarity edges.

## Evidence

- [abstract] MetFusion scoring used to create the weight function of the combined graph: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function, producing a"
- [abstract] Random walk over the weighted graph from seed nodes: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [methods] ChemWalker uses MetFrag for in silico annotation and the Fusion concept for ranking improvement from spectral library: "ChemWalker uses MetFrag for in silico annotation"
- [readme] pyrwr RWR requires positively weighted networks: "Note that 
* RWR is defined on positively weighted networks; thus, only positive weights are allowed."
- [readme] Redundant weighted edges are summed in the final adjacency matrix: [evidence span withheld — anchor status: external_doc]
