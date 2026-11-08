---
name: random-walk-seed-node-analysis
description: Use when annotating GNPS molecular networks where only ~10% of nodes
  have spectral library matches and you need annotations on nodes lacking a direct
  edge to any matched seed node.
when_to_use_negative:
- Every target node already has a direct neighbour with a spectral library match,
  so NAP-style local first-neighbour propagation suffices
- No candidate structures are available from in silico fragmentation (ChemWalker relies
  on MetFrag combinatorial fragmentation results)
- The graph contains non-positive edge weights — pyrwr's RWR is defined only on positively
  weighted networks
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ChemWalker
  role: Python package that orchestrates the random-walk annotation propagation on
    the MetFusion-weighted molecular network
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Computes random walk with restart score vectors from seed nodes via power
    iteration (numpy/scipy, optional GPU via pytorch)
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFusion
  role: Scoring function used to create the weight function combining spectral-network
    edges and structural similarity among candidate structures
- name: MetFrag
  role: Performs combinatorial in silico fragmentation of candidate structures searched
    in biologically relevant databases
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: Python
  role: Runtime environment for ChemWalker; install in a dedicated conda environment
    from environment.yml
  repo: https://github.com/computational-chemical-biology/ChemWalker
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:c679e9e57fda216dc259d59ebbbd3eecb041d21d3251ae953756ed27986d45c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - Python
  - pyrwr
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# random-walk-seed-node-analysis

## Summary

Propagate spectral library match identities across a molecular network using random walk with restart (pyrwr) on a MetFusion-weighted graph, so that nodes with no direct neighbour matching a spectral library still receive candidate annotations. This extends annotation propagation beyond NAP's local first-neighbour limitation.

## When to use

Use when annotating GNPS molecular networks where only ~10% of nodes have spectral library matches and you need annotations on nodes lacking a direct edge to any matched seed node.

## When NOT to use

- Every target node already has a direct neighbour with a spectral library match, so NAP-style local first-neighbour propagation suffices
- No candidate structures are available from in silico fragmentation (ChemWalker relies on MetFrag combinatorial fragmentation results)
- The graph contains non-positive edge weights — pyrwr's RWR is defined only on positively weighted networks

## Inputs

- Molecular network (spectral network edge list) from GNPS, with nodes representing mass signals
- Per-spectrum candidate annotation tables from MetFrag in silico fragmentation against biologically relevant databases
- Seed-node list of spectral library matches (<= 10% of nodes in the benchmark regime)
- MetFrag CL output (tested with MetFrag2.3.-CL.jar)

## Outputs

- Random-walk candidate probability scores per node (RWR score vector from pyrwr)
- Ranked candidate annotations for every network node, including non-neighbor nodes
- Aggregated per-spectrum non-neighbor annotation counts (e.g. annotation_propagation_nonneighbor_summary.csv)

## How to apply

Load the molecular network (e.g. the NIST 555-spectra benchmark) together with per-spectrum candidate annotation tables produced by MetFrag in silico fragmentation. Designate seed nodes as those with spectral library matches (at most 10% of nodes in the benchmark regime). Build a weighted graph that combines spectral-network edges with structural similarity among candidate structures, using the MetFusion scoring function as the weight. Run a random walk with restart from the seed nodes over this weighted graph via the pyrwr solver (RWR/PPR power-iteration implementation in numpy/scipy) to compute candidate probabilities. Finally, for each spectrum, count annotations landing on nodes without a direct edge to any spectral-library-match node to confirm propagation reached beyond the first neighbour.

## Related tools

- **ChemWalker** (Python package that orchestrates the random-walk annotation propagation on the MetFusion-weighted molecular network) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Computes random walk with restart score vectors from seed nodes via power iteration (numpy/scipy, optional GPU via pytorch)) — https://github.com/jinhongjung/pyrwr
- **MetFusion** (Scoring function used to create the weight function combining spectral-network edges and structural similarity among candidate structures)
- **MetFrag** (Performs combinatorial in silico fragmentation of candidate structures searched in biologically relevant databases) — https://github.com/ipb-halle/MetFragRelaunched
- **Python** (Runtime environment for ChemWalker; install in a dedicated conda environment from environment.yml) — https://github.com/computational-chemical-biology/ChemWalker

## Examples

```
from pyrwr.ppr import PPR; ppr = PPR(); ppr.read_graph(input_graph, 'undirected'); r = ppr.compute(seeds, c=0.15, epsilon=1e-9, max_iters=100)
```

## Evaluation signals

- Success requires a nonzero count of annotations recorded on nodes not directly connected to any spectral-library-match node, reproducing the paper's claim that propagation reaches beyond the first neighbour
- Benchmark regime holds at most 10% of network nodes designated as seed nodes with spectral library matches
- RWR score vector shape must be (n, 1) over the weighted graph's nodes, with the seed node receiving the highest restart probability (pyrwr default restart probability c=0.15, epsilon=1e-9, max_iters=100)
- Compare against NAP: annotations should appear on nodes NAP would leave unannotated due to its local first-neighbour limitation
- ChemWalker should run in less time and scale better than NAP (improvements in running time, scalability and maintainability)

## Limitations

- One limitation of NAP is that information for the spectral matches is only propagated locally to the first neighbour — this skill exists specifically to overcome that, but it depends on candidate structures from MetFrag fragmentation
- pyrwr RWR requires positively weighted networks; redundant edges are summed and deadends must be handled (handles-deadend option, default True)
- Annotation of mass signals remains the biggest bottleneck in untargeted mass spectrometry of complex mixtures — random-walk propagation produces probabilistic candidates, not definitive identifications

## Evidence

- [abstract] Propagation beyond first neighbour: "Here, we show that annotation propagation can be expanded to nodes not directly connected to spectral matches using random walks on graphs"
- [abstract] NAP local limitation: "One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match"
- [abstract] Weighted graph construction: "we have used MetFusion Scoring function to create a weight function, producing a weighted graph"
- [abstract] Random walk from seeds: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [intro] pyrwr usage: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]"
- [readme] pyrwr RWR score shape: [evidence span withheld — anchor status: not_found]
- [readme] Positive weights required: [evidence span withheld — anchor status: external_doc]
