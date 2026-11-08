---
name: spectral-network-connected-component-traversal
description: Use when a GNPS-style molecular network has MetFrag candidate structures
  per node, spectral library matches are available as seeds, and you must annotate
  nodes beyond the first neighbors of those matches.
when_to_use_negative:
- No MetFrag candidate structures are available — ChemWalker's graph requires structural
  similarity among in silico fragmentation candidates, not just a spectral network
- No spectral library matches exist to serve as seed nodes — the walk departs from
  spectral-match seeds and cannot start without them
- The goal is direct spectral library matching or first-neighbor propagation (NAP-style
  local propagation) rather than propagation to distant nodes via random walks
tools:
- name: ChemWalker
  role: Python package orchestrating graph construction, connected-component splitting,
    seed marking, and random-walk-based annotation propagation
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Computes the random-walk-with-restart / personalized PageRank score vectors
    on each component seeded at the spectral-match nodes
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Provides the combinatorial in silico fragmentation results and candidate structures
    whose pairwise similarity forms the weighted graph edges
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Scoring function used to create the weight function producing the weighted
    graph walked by pyrwr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:c10f7aff61789a11c63625c8a07d6755307104d41a11a17e3dabe5f3beb860af
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# spectral-network-connected-component-traversal

## Summary

Propagates spectral library match annotations across molecular networks by splitting the weighted graph (spectral network + MetFusion-weighted structural similarity among MetFrag candidates) into connected components and running a random walk with restart (pyrwr) from seed nodes, so annotations reach nodes not directly connected to a spectral match — the key limitation of NAP.

## When to use

Use when a GNPS-style molecular network has MetFrag candidate structures per node, spectral library matches are available as seeds, and you must annotate nodes beyond the first neighbors of those matches.

## When NOT to use

- No MetFrag candidate structures are available — ChemWalker's graph requires structural similarity among in silico fragmentation candidates, not just a spectral network
- No spectral library matches exist to serve as seed nodes — the walk departs from spectral-match seeds and cannot start without them
- The goal is direct spectral library matching or first-neighbor propagation (NAP-style local propagation) rather than propagation to distant nodes via random walks

## Inputs

- Weighted graph combining the spectral network and structural-similarity edges among MetFrag candidate structures (MetFusion weight function), as adjacency/edge-list table
- List of seed node identifiers (spectral library matches), ≤10% of all nodes
- MetFrag in silico combinatorial fragmentation results of candidate structures against biologically relevant databases

## Outputs

- probabilities.csv — walking probability of every candidate-structure node per connected component, keyed by node identifier
- Per-component random-walk score vectors (pyrwr output: node id + score)

## How to apply

Load the weighted graph — spectral network edges combined with structural-similarity edges among MetFrag candidate structures scored by the MetFusion weight function — as an adjacency/edge-list table with pandas. Mark spectral library match nodes as seeds, enforcing that seeds constitute at most 10% of all nodes. Split the graph into connected components and, on each component, run a random walk with restart seeded at the spectral-match nodes using pyrwr (pyrwr supports PPR with multiple seeds and requires positively weighted networks). Collect the walking probability of every candidate-structure node per component into a single table keyed by node identifier, then write the probability table to probabilities.csv. Rationale: random-walk probabilities decay with graph distance, so information propagates through the component's candidate-structure similarity edges rather than stopping at first neighbors.

## Related tools

- **ChemWalker** (Python package orchestrating graph construction, connected-component splitting, seed marking, and random-walk-based annotation propagation) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Computes the random-walk-with-restart / personalized PageRank score vectors on each component seeded at the spectral-match nodes) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Provides the combinatorial in silico fragmentation results and candidate structures whose pairwise similarity forms the weighted graph edges) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Scoring function used to create the weight function producing the weighted graph walked by pyrwr)

## Examples

```
pyrwr --query-type rwr --graph-type undirected --input-path component_edges.tsv --output-path output/scores.tsv --seeds 10982 --device cpu
```

## Evaluation signals

- Seed (spectral library match) nodes constitute at most 10% of all nodes in the graph
- Every candidate-structure node in each connected component receives a walking probability in the final table
- Probabilities are non-negative and decay with distance from spectral-match seeds within a component (higher for candidates connected to seeds via high MetFusion-weight similarity edges)
- probabilities.csv is keyed by node identifier and covers all components jointly
- Annotated nodes include ones not directly connected to a spectral match, demonstrating propagation beyond the NAP first-neighbor limitation

## Limitations

- pyrwr requires positively weighted networks — only positive edge weights are allowed, and redundant edges in the input have their weights summed
- pyrwr RWR accepts only a single seed; multiple seeds require the PPR query type, and node ids must be non-negative integers
- Propagation quality depends on MetFrag candidate structures and the MetFusion weighting; nodes in components with no spectral-match seeds receive no propagated annotation

## Evidence

- [abstract] Annotation propagation can be expanded to nodes not directly connected to spectral matches using random walks on graphs: [evidence span withheld — anchor status: not_found]
- [abstract] random walk over weighted graph from seed nodes: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes"
- [intro] ChemWalker is a python package to propagate spectral library match identities: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation"
- [readme] pyrwr implements RWR via power iteration: [evidence span withheld — anchor status: not_found]
