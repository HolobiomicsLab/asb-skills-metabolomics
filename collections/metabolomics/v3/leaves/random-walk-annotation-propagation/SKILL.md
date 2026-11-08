---
name: random-walk-annotation-propagation
description: Use when molecular networking (e.g. GNPS) leaves most nodes unannotated
  because NAP only propagates library matches to first neighbors, and you need walking
  probabilities for candidate structures not directly connected to any spectral match.
when_to_use_negative:
- Nodes of interest are direct first neighbors of spectral library matches — NAP's
  local propagation is sufficient and the random-walk machinery is unnecessary.
- The graph contains non-positive edge weights — RWR in pyrwr is defined only on positively
  weighted networks.
- Seed nodes exceed 10% of all nodes — propagation from such a large seed fraction
  is outside ChemWalker's tested regime.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: ChemWalker
  role: Python package that builds the MetFusion-weighted graph and orchestrates random-walk
    propagation of spectral library match identities through MetFrag candidate structures
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Computes the random walk with restart / personalized PageRank scores over
    each connected component from the spectral-match seed nodes
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Provides combinatorial in silico fragmentation results (candidate structures)
    searched against biologically relevant databases; tested with MetFrag2.3-CL.jar
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Scoring function whose weight function combines the spectral network with
    structural similarity among candidate structures into the weighted graph
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:32653b193d18b4409e9733cbf51718d2f43375b8b4305c37a6a0befb68ebfaaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - Network Annotation Propagation (NAP)
  - RDKit
  - pyrwr
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# random-walk-annotation-propagation

## Summary

Propagate spectral library match annotations across molecular networks beyond first neighbors using ChemWalker: a MetFusion-weighted graph of the spectral network plus structural similarity among MetFrag candidate structures is submitted to random walk with restart (pyrwr) from seed nodes.

## When to use

Use when molecular networking (e.g. GNPS) leaves most nodes unannotated because NAP only propagates library matches to first neighbors, and you need walking probabilities for candidate structures not directly connected to any spectral match.

## When NOT to use

- Nodes of interest are direct first neighbors of spectral library matches — NAP's local propagation is sufficient and the random-walk machinery is unnecessary.
- The graph contains non-positive edge weights — RWR in pyrwr is defined only on positively weighted networks.
- Seed nodes exceed 10% of all nodes — propagation from such a large seed fraction is outside ChemWalker's tested regime.

## Inputs

- Weighted graph combining the spectral network and structural-similarity edges scored by the MetFusion weight function, as an adjacency/edge-list table loadable with pandas
- Spectral library match nodes designated as seeds (<=10% of all nodes)
- MetFrag 2.3 command-line (MetFrag2.3-CL.jar) in silico fragmentation results over candidate structures from biologically relevant databases

## Outputs

- Walking probability for every candidate-structure node per connected component, aggregated into a single table keyed by node identifier (e.g. probabilities.csv)

## How to apply

Build a weighted graph combining the spectral network with structural similarity among MetFrag candidate structures, using the MetFusion scoring function as the weight function; only positive weights are valid since RWR requires positively weighted networks. Mark spectral library match nodes as seeds (seeds must constitute at most 10% of all nodes), then split the graph into connected components. For each component, run random walk with restart via pyrwr's PPR (multiple-seed) routine seeded at the spectral-match nodes, using power iteration parameters such as restart probability c=0.15 and error tolerance epsilon=1e-9. Collect the walking probability of every candidate-structure node per component into a single table keyed by node identifier and export it (e.g. probabilities.csv). Rationale: departing from the combination of a spectral network and structural similarity, the random walk calculates the probability of 'walking' through a set of candidates, allowing information to reach nodes not directly connected to the spectral library match.

## Related tools

- **ChemWalker** (Python package that builds the MetFusion-weighted graph and orchestrates random-walk propagation of spectral library match identities through MetFrag candidate structures) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Computes the random walk with restart / personalized PageRank scores over each connected component from the spectral-match seed nodes) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Provides combinatorial in silico fragmentation results (candidate structures) searched against biologically relevant databases; tested with MetFrag2.3-CL.jar) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Scoring function whose weight function combines the spectral network with structural similarity among candidate structures into the weighted graph)

## Examples

```
from pyrwr.ppr import PPR; ppr = PPR(); ppr.read_graph(input_graph, graph_type); r = ppr.compute(seeds, c, epsilon, max_iters)
```

## Evaluation signals

- Seeds (spectral library match nodes) constitute at most 10% of all nodes in each component before running the walk.
- Every candidate-structure node in each connected component receives a finite, positive walking-probability value in the output table; the table is keyed by node identifier with no missing components.
- Seed nodes should recover the highest walking probabilities relative to non-seed nodes in their component, reflecting restart at spectral matches.
- All edge weights in the input graph are positive; redundant edges must have been summed (e.g. weights 3 and 5 collapse to 8) per pyrwr's input rules.
- On the NIST 555-spectra benchmark, top-10 candidate classification should be statistically indistinguishable from NAP (per the paper's supplementary equivalence result).

## Limitations

- ChemWalker was tested on MetFrag2.3-CL.jar; other MetFrag versions/releases may require validation.
- pyrwr only supports positive edge weights and sums redundant edges, so graph construction must respect these constraints.
- RWR with a single seed allows only one seed node; propagation from multiple spectral-match nodes requires the PPR query type, and nodes with no path to a seed receive no propagated information.

## Evidence

- [abstract] Annotation propagation can be expanded to nodes not directly connected to spectral matches using random walks on graphs: "Here, we show that annotation propagation can be expanded to nodes not directly connected to spectral matches using random walks on graphs"
- [abstract] MetFusion weight function and random walk over weighted graph from seed nodes: "we have used MetFusion Scoring function to create a weight function, producing a weighted graph"
- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by in silico fragmentation: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]"
- [other] Seeds at most 10% of nodes; connected components; pyrwr walk; probabilities.csv: [evidence span withheld — anchor status: not_found]
- [readme] pyrwr only supports positive edge weights and sums redundant edges: [evidence span withheld — anchor status: external_doc]
- [readme] ChemWalker installation via conda environment and pip from GitHub: "conda env create -f environment.yml
conda activate chemwalker 
pip install git+https://github.com/computational-chemical-biology/ChemWalker.git"
