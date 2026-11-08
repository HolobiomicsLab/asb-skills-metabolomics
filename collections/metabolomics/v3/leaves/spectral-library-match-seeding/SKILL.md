---
name: spectral-library-match-seeding
description: Use when a molecular network has spectral library matches available for
  a subset of nodes (seeds) and per-spectrum MetFrag candidate lists, and annotations
  must be propagated to candidate structures beyond the first neighbor of a match.
when_to_use_negative:
- No spectral library matches exist for the network, so there are no seed nodes to
  anchor the random walk
- Only local, first-neighbor propagation is needed and the NAP local-propagation behavior
  is sufficient
- The graph contains zero or negative edge weights — pyrwr requires positively weighted
  networks for RWR
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ChemWalker
  role: Python package that defines the seed nodes from spectral library matches and
    orchestrates propagation of those identities through MetFrag candidate structures
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Computes the random walk with restart (single seed) or Personalized PageRank
    (multiple seeds) from the spectral-library-match seed nodes over the weighted
    graph
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFrag
  role: Produces the in silico combinatorial fragmentation candidate structures that
    the seeded random walk annotates
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Provides the scoring function that weights the graph edges combining the spectral
    network with structural similarity among candidate structures
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:f7767a2d59fa05ffc3931bc89a393a3f96b0873ea7266eb927b187b93c256175
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - RDKit
  - pyrwr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# spectral-library-match-seeding

## Summary

Designates nodes with spectral library matches as seed nodes and runs random-walk-with-restart propagation over a MetFusion-weighted graph (ChemWalker) to transfer identities to candidate structures that lack direct spectral matches.

## When to use

Use when a molecular network has spectral library matches available for a subset of nodes (seeds) and per-spectrum MetFrag candidate lists, and annotations must be propagated to candidate structures beyond the first neighbor of a match.

## When NOT to use

- No spectral library matches exist for the network, so there are no seed nodes to anchor the random walk
- Only local, first-neighbor propagation is needed and the NAP local-propagation behavior is sufficient
- The graph contains zero or negative edge weights — pyrwr requires positively weighted networks for RWR

## Inputs

- Molecular (spectral) network of MS signals with edges between related spectra
- Per-spectrum MetFrag in silico candidate structure lists (from MetFrag2.3-CL.jar)
- Spectral library match table identifying which network nodes have seed identities (seed nodes, up to 10% of nodes in the NIST 555-spectra benchmark regime)
- MetFusion-weighted graph as a pyrwr edge-list file (source \t target \t positive weight)

## Outputs

- Per-seed RWR (or multi-seed PPR) probability vector assigning a walking probability to every candidate node
- Ranked candidate annotation per spectrum (top-10 candidate classification)
- Per-spectrum summary tables such as counts of annotations on nodes not directly connected to a spectral-library-match node

## How to apply

Designate seed nodes from spectral library matches — in the NIST 555-spectra benchmark this regime keeps seed nodes to at most 10% of network nodes. Build a weighted graph that combines the spectral network with structural similarity among candidate structures using the MetFusion scoring function, so edges encode both MS2 spectral relations and structural similarity. Run the random walk with the pyrwr solver from the seed nodes over this weighted graph to compute the probability of 'walking' through each candidate; note that plain RWR allows a single seed, so use pyrwr's Personalized PageRank (PPR) query when multiple seed nodes must be combined. Rank candidates per spectrum by the resulting probabilities and evaluate top-10 candidate classification against the ChemWalker-versus-NAP benchmark, where the key expected gain over NAP is that information reaches nodes not directly connected to the spectral library match.

## Related tools

- **ChemWalker** (Python package that defines the seed nodes from spectral library matches and orchestrates propagation of those identities through MetFrag candidate structures) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Computes the random walk with restart (single seed) or Personalized PageRank (multiple seeds) from the spectral-library-match seed nodes over the weighted graph) — https://github.com/jinhongjung/pyrwr
- **MetFrag** (Produces the in silico combinatorial fragmentation candidate structures that the seeded random walk annotates) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Provides the scoring function that weights the graph edges combining the spectral network with structural similarity among candidate structures)

## Examples

```
from pyrwr.ppr import PPR; ppr = PPR(); ppr.read_graph(input_graph, graph_type); r = ppr.compute(seeds, c, epsilon, max_iters)
```

## Evaluation signals

- Validation succeeds when a nonzero count of annotations is recorded on nodes not directly connected to the spectral library match, reproducing the paper's claim that propagation reaches nodes beyond the first neighbor
- Seed nodes retain their spectral-library-match identity (walk starts at the seeds); benchmark regime keeps seed nodes at or below 10% of network nodes
- pyrwr output rows follow the node-id \t RWR-score format, with scores non-negative and consistent with a restart probability (default c = 0.15) and epsilon = 1e-9 convergence
- Per-fingerprint / per-seed ranking results aggregate cleanly into a CSV (e.g. fingerprint_benchmark_results.csv) with a defined default-fingerprint baseline
- Top-10 candidate classification on the NIST 555-spectra benchmark is compared against the NAP benchmark rather than reported in isolation

## Limitations

- pyrwr RWR allows only a single seed; combining multiple spectral-library-match seeds requires the PPR query type instead
- RWR is defined only on positively weighted networks, so graph weights must all be positive before running the walk
- The reference comparison baseline (NAP) propagates information only locally to the first neighbor of a spectral match, so ChemWalker's advantage is demonstrated specifically for annotations beyond the first neighbor
- ChemWalker's benchmark equivalence with NAP is assessed on the NIST 555-spectra top-10 classification; on other benchmarks the result is not guaranteed to transfer

## Evidence

- [abstract] This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches): "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [methods] ChemWalker uses the Fusion concept, proposed on MetFusion, for ranking improvement from spectral library.: "ChemWalker uses the Fusion concept, proposed on MetFusion, for ranking improvement from spectral library."
- [abstract] One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match: "One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match"
- [readme] Besides RWR, `pyrwr` supports computing Personalized PageRank (PPR) with multiple seeds and PageRank which are well-known variants of RWR.: [evidence span withheld — anchor status: external_doc]
- [readme] RWR is defined on positively weighted networks; thus, only positive weights are allowed.: [evidence span withheld — anchor status: external_doc]
