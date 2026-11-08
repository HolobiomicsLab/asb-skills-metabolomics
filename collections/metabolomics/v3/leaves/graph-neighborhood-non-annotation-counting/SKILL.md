---
name: graph-neighborhood-non-annotation-counting
description: Use when benchmarking an annotation-propagation method (e.g. ChemWalker
  vs NAP) and you need to prove that annotations reach network nodes lacking any direct
  neighbour with a spectral library match.
when_to_use_negative:
- When the goal is to verify only first-neighbour propagation (e.g. NAP's local regime)
  — the count would be zero by construction and uninformative
- When the network has no spectral library matches at all (no seeds), since the count
  is defined relative to seed-node neighbourhoods
- When evaluating annotation quality/accuracy against ground truth rather than propagation
  reach — this skill measures where annotations land, not whether they are correct
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ChemWalker
  role: Random-walk propagation method whose output annotations are audited for non-neighbour
    reach
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: pyrwr
  role: Python random-walk-with-restart solver backing ChemWalker's candidate probability
    computation
  repo: https://github.com/jinhongjung/pyrwr
- name: MetFusion
  role: Scoring function used to build the weighted graph over which propagation and
    reach are measured
- name: Network Annotation Propagation (NAP)
  role: Baseline method limited to first-neighbour propagation; the non-neighbour
    count distinguishes ChemWalker from it
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:9b9a03c498d1b388f8054533ffe82bad11b428f3793a9f9e546cbdae68777320
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFusion
  - Network Annotation Propagation (NAP)
  - pyrwr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# graph-neighborhood-non-annotation-counting

## Summary

A validation technique for annotation-propagation methods on molecular networks: it quantifies annotations landing on nodes that have no direct edge to any spectral-library-match (seed) node, demonstrating propagation beyond NAP's local first-neighbour limitation.

## When to use

Use when benchmarking an annotation-propagation method (e.g. ChemWalker vs NAP) and you need to prove that annotations reach network nodes lacking any direct neighbour with a spectral library match.

## When NOT to use

- When the goal is to verify only first-neighbour propagation (e.g. NAP's local regime) — the count would be zero by construction and uninformative
- When the network has no spectral library matches at all (no seeds), since the count is defined relative to seed-node neighbourhoods
- When evaluating annotation quality/accuracy against ground truth rather than propagation reach — this skill measures where annotations land, not whether they are correct

## Inputs

- Molecular network (spectral network graph, e.g. NIST 555-spectra benchmark)
- Per-spectrum candidate annotation tables from in silico fragmentation (MetFrag results)
- Set of seed node IDs defined by spectral library matches (≤10% of nodes)
- MetFusion-weighted graph combining spectral-network edges with structural similarity among candidate structures

## Outputs

- Per-spectrum counts of annotations on nodes lacking a direct edge to any spectral-library-match node
- Aggregated summary CSV (annotation_propagation_nonneighbor_summary.csv)

## How to apply

Load the benchmark molecular network (e.g. the NIST 555-spectra network) with its per-spectrum candidate annotation tables in Python. Designate seed nodes as spectral library matches, keeping seeds to at most 10% of network nodes to match the benchmark regime. Run the propagation method over the MetFusion-weighted graph, which combines spectral-network edges with structural similarity among MetFrag candidate structures, computing candidate probabilities via the pyrwr-backed random walk from seeds. Then, for each spectrum, inspect the top propagated annotations and count those assigned to nodes with no direct edge to any seed node. Aggregate these non-neighbour annotation counts across all 555 spectra into a summary table (e.g. annotation_propagation_nonneighbor_summary.csv) and check the total is nonzero.

## Related tools

- **ChemWalker** (Random-walk propagation method whose output annotations are audited for non-neighbour reach) — https://github.com/computational-chemical-biology/ChemWalker
- **pyrwr** (Python random-walk-with-restart solver backing ChemWalker's candidate probability computation) — https://github.com/jinhongjung/pyrwr
- **MetFusion** (Scoring function used to build the weighted graph over which propagation and reach are measured)
- **Network Annotation Propagation (NAP)** (Baseline method limited to first-neighbour propagation; the non-neighbour count distinguishes ChemWalker from it)

## Examples

```
conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- Nonzero total count of annotations assigned to nodes not directly connected to any spectral-library-match node across the 555-spectra benchmark
- Seed fraction does not exceed 10% of network nodes (benchmark regime)
- Per-spectrum counts exist for all 555 spectra in the summary CSV (no missing rows)
- Network Analysis of NAP under the same seeds yields zero non-neighbour annotations, so the measured reach is attributable to the random-walk propagation

## Limitations

- NAP's own limitation means the first-neighbour baseline cannot produce non-neighbour annotations, so comparison relies on the propagated method alone
- Counting non-neighbour annotations measures propagation reach, not annotation correctness — a nonzero count does not imply accurate annotations
- The result depends on the MetFusion-weighted graph construction; changes to edge weights alter which nodes the walk can reach

## Evidence

- [abstract] Annotation propagation can be expanded to nodes not directly connected to spectral matches using random walks on graphs: [evidence span withheld — anchor status: not_found]
- [abstract] NAP first-neighbour limitation: "One of the limitations of NAP is that the information for the spectral matches is only propagated locally, to the first neighbor of a spectral match"
- [abstract] Random walk departing from seed nodes: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [readme] pyrwr random walk dependency: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random"
