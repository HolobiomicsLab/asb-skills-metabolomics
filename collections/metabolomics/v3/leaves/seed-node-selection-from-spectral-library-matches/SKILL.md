---
name: seed-node-selection-from-spectral-library-matches
description: Use when preparing a weighted molecular network (spectral network + MetFusion-scored
  structural similarity edges) for random-walk propagation and you need to designate
  GNPS spectral library match nodes as restart seeds.
when_to_use_negative:
- The input is raw mass spectrometry data or unfragmented spectra — in silico annotation
  with MetFrag and graph construction must happen first
- Spectral library matches cover more than 10% of network nodes, violating the seed
  sparsity constraint the method was designed under
- You need to propagate annotations only to first neighbors of spectral matches —
  NAP already does this without a random walk
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ChemWalker
  role: Python library whose workflow marks spectral library match nodes as seeds
    (at most 10% of nodes) and runs the random walk per connected component
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Provides the random-walk-with-restart / PPR computation seeded at the spectral-match
    nodes; PPR supports multiple seeds
  repo: https://github.com/jinhongjung/pyrwr
- name: Python
  role: Implementation language; seed nodes are marked from pandas-loaded edge/adjacency
    tables before the walk
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:538d4ab368e9d4dfff9466a76dc3bccdaef371b2d943ca00022d8b6c1c8d132d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - Python
  - pyrwr
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# seed-node-selection-from-spectral-library-matches

## Summary

Selects seed nodes for ChemWalker's random-walk annotation propagation by marking spectral library match nodes within each molecular network component, with the constraint that seeds constitute at most 10% of all nodes.

## When to use

Use when preparing a weighted molecular network (spectral network + MetFusion-scored structural similarity edges) for random-walk propagation and you need to designate GNPS spectral library match nodes as restart seeds.

## When NOT to use

- The input is raw mass spectrometry data or unfragmented spectra — in silico annotation with MetFrag and graph construction must happen first
- Spectral library matches cover more than 10% of network nodes, violating the seed sparsity constraint the method was designed under
- You need to propagate annotations only to first neighbors of spectral matches — NAP already does this without a random walk

## Inputs

- Weighted graph (adjacency/edge-list table) combining the spectral network and MetFusion-scored structural similarity among MetFrag candidate structures
- Spectral library match annotations identifying which nodes are GNPS spectral library matches

## Outputs

- Seed node list (per connected component) used to seed the pyrwr random-walk-with-restart computation
- Walking probability table for every candidate-structure node per component (probabilities.csv)

## How to apply

Load the weighted graph, built upstream from the spectral network plus structural-similarity edges scored by the MetFusion weight function, as an adjacency/edge-list table with pandas. Identify the nodes whose candidate structures were matched against a spectral library (GNPS spectral library matches) and mark them as seeds. Enforce the constraint that seeds make up at most 10% of all nodes in the graph, since the walk quality depends on a sparse, high-confidence seed set. After splitting the graph into connected components, the seeds act as the restart set for the pyrwr random-walk-with-restart routine (PPR with multiple seeds) run per component. Walking probabilities computed from these seeds are then collected per candidate-structure node and written to probabilities.csv.

## Related tools

- **ChemWalker** (Python library whose workflow marks spectral library match nodes as seeds (at most 10% of nodes) and runs the random walk per connected component) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Provides the random-walk-with-restart / PPR computation seeded at the spectral-match nodes; PPR supports multiple seeds) — https://github.com/jinhongjung/pyrwr
- **Python** (Implementation language; seed nodes are marked from pandas-loaded edge/adjacency tables before the walk)

## Examples

```
from pyrwr.ppr import PPR; ppr = PPR(); ppr.read_graph(input_graph, 'undirected'); r = ppr.compute(seeds, c=0.15, epsilon=1e-9, max_iters=100)
```

## Evaluation signals

- Seed count per component is at most 10% of that component's nodes
- Every seed node corresponds to a node with a GNPS spectral library match in the input annotations
- Random-walk scores are computed for every candidate-structure node in each connected component, not only nodes adjacent to seeds
- Walking probabilities are strictly positive and finite for all reachable nodes, and the aggregated table is keyed by node identifier
- Output probabilities.csv contains one probability value per candidate-structure node per component

## Limitations

- Seed selection is capped at 10% of nodes; components with many spectral library matches exceed the design constraint of the method
- The walk can only start from nodes with spectral library matches; components lacking any match receive no propagation
- RWR/PPR in pyrwr requires positively weighted networks — edge weights from the MetFusion weight function must be positive

## Evidence

- [methods] Mark spectral library match nodes as seeds, enforcing that seeds constitute at most 10% of all nodes.: [evidence span withheld — anchor status: not_found]
- [abstract] random walk over the weighted graph from seed nodes (spectral library matches): "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [readme] pyrwr PPR supports multiple seeds: [evidence span withheld — anchor status: external_doc]
- [readme] RWR defined on positively weighted networks: [evidence span withheld — anchor status: external_doc]
- [methods] seeds are the spectral-match nodes in ChemWalker: [evidence span withheld — anchor status: not_found]
