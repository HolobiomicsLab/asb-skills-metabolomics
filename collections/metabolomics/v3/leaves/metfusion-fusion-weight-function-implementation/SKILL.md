---
name: metfusion-fusion-weight-function-implementation
description: 'Use when building the weighted molecular-network graph for ChemWalker
  propagation: you have MetFrag in silico fragmentation results for candidates of
  each network node, pairwise spectral cosine scores from library matching or network
  edges, and structural (chemical) similarity between candidate.'
when_to_use_negative:
- The graph edges are already weighted by a different scheme and you only need to
  run the random walk; this skill is specifically the MetFusion-based weight construction
  step.
- You lack MetFrag candidate scores or structural similarity between candidate pairs
  — the weight function requires all three signal sources (MetFrag score, spectral
  cosine, chemical similarity) to combine.
- The goal is NAP-style local propagation only to first neighbors of spectral matches;
  ChemWalker's weighted-graph random walk is intended to expand propagation beyond
  directly connected nodes.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
tools:
- name: ChemWalker
  role: Python package that implements the MetFusion-derived weight function, builds
    the weighted graphml graph, and runs the random walk over it
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: MetFrag
  role: Produces the in silico combinatorial fragmentation candidate scores that are
    alpha-averaged in the weight function
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFusion
  role: Source of the Fusion scoring/weight concept adapted by ChemWalker for ranking
    improvement from the spectral library
- name: pyrwr
  role: Consumes the weighted graph to compute the random walk with restart from seed
    nodes (spectral library matches)
  repo: https://github.com/jinhongjung/pyrwr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:0acaae428bf29ff5a202c5ca1eddad6e46a712af0372a61db2a5afca22f7ea67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - MetFrag
  - MetFusion
  - pyrwr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metfusion-fusion-weight-function-implementation

## Summary

Implementation of the MetFusion-derived Fusion weight function that combines MetFrag candidate scores, spectral cosine similarity, and chemical similarity of candidate structures into edge weights for the ChemWalker weighted graph. This weighted graphml is the substrate the ChemWalker random walk operates on, so correct edge weighting determines which annotations propagate.

## When to use

Use when building the weighted molecular-network graph for ChemWalker propagation: you have MetFrag in silico fragmentation results for candidates of each network node, pairwise spectral cosine scores from library matching or network edges, and structural (chemical) similarity between candidate pairs, and need to fuse them into edge weights.

## When NOT to use

- The graph edges are already weighted by a different scheme and you only need to run the random walk; this skill is specifically the MetFusion-based weight construction step.
- You lack MetFrag candidate scores or structural similarity between candidate pairs — the weight function requires all three signal sources (MetFrag score, spectral cosine, chemical similarity) to combine.
- The goal is NAP-style local propagation only to first neighbors of spectral matches; ChemWalker's weighted-graph random walk is intended to expand propagation beyond directly connected nodes.

## Inputs

- MetFrag in silico combinatorial fragmentation results (per-candidate annotation scores, e.g. from MetFrag2.3-CL.jar runs over biologically relevant databases)
- Spectral cosine similarity scores between connected molecular-network nodes (from spectral library matching / GNPS molecular network)
- Chemical (structural) similarity values among candidate structures
- Alpha fusion parameter

## Outputs

- Weighted graph in graphml format where edge weights w_ij follow the MetFusion-style formulation
- Candidate probabilities from the random walk over the weighted graph, departing from seed nodes represented by spectral library matches

## How to apply

Compute per-node MetFrag annotation scores for all candidate structures, then for each edge between molecular-network nodes i and j form the MetFusion-style MetFrag term: an alpha-weighted average of the two nodes' candidate scores. Combine this with a second term, weight 1-alpha, consisting of the spectral cosine score multiplied by the chemical similarity between the candidate structures of i and j, passed through a sigmoid to yield the edge weight w_ij. The alpha parameter is the decision point controlling how much the propagation trust rests on standalone MetFrag scores versus the combined spectral-cosine and structural-similarity signal. Assemble nodes and computed edge weights into a graphml-formatted weighted graph, then run pyrwr random walk with restart on it, departing from seed nodes (spectral library matches). Rationale: the Fusion concept, proposed in MetFusion, is used by ChemWalker for ranking improvement from spectral library, so the weight function encodes both in silico fragmentation support and spectral/structural concordance between neighboring nodes.

## Related tools

- **ChemWalker** (Python package that implements the MetFusion-derived weight function, builds the weighted graphml graph, and runs the random walk over it) — https://github.com/computational-chemical-biology/ChemWalker
- **MetFrag** (Produces the in silico combinatorial fragmentation candidate scores that are alpha-averaged in the weight function) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFusion** (Source of the Fusion scoring/weight concept adapted by ChemWalker for ranking improvement from the spectral library)
- **pyrwr** (Consumes the weighted graph to compute the random walk with restart from seed nodes (spectral library matches)) — https://github.com/jinhongjung/pyrwr

## Evaluation signals

- The output weighted graph loads as a valid graphml file and every edge carries a finite, non-negative weight w_ij (RWR is defined on positively weighted networks, so zero/negative/NaN weights are invalid).
- Edge weights reflect the MetFusion formulation: the alpha-weighted average of MetFrag scores plus (1-alpha) * sigmoid(cosine * chemical similarity); perturbing alpha shifts the balance between fragmentation and spectral/structural terms predictably.
- Graph node set matches the molecular-network nodes; seeds passed to the random walk correspond to nodes with spectral library matches.
- Downstream pyrwr run converges (within epsilon/max-iters) and produces a probability vector over candidates; nodes not directly connected to spectral matches receive non-zero propagated annotation probabilities.
- ChemWalker used for the Fusion weight construction matches the paper's implementation — verify against the ChemWalker repository / basic_use_colab notebook

## Limitations

- The paper evaluates this mechanism only as part of the end-to-end random-walk propagation rather than through a standalone quantitative criterion, so correctness is judged on graph validity and downstream propagation quality.
- RWR (pyrwr) is defined on positively weighted networks; only positive edge weights are allowed, and redundant weighted edges would be summed if using pyrwr's tsv input path directly.
- The weight function depends on the alpha parameter; an unsuitable choice skews propagation between MetFrag-score dominance and spectral/structural-similarity dominance.
- The information propagated depends on MetFrag candidate quality: chemically uninformative candidate sets yield weak fusion weights regardless of spectral similarity.
- pyrwr was originally the random-walk backend in the intro description; version drift between pyrwr and ChemWalker may require verifying the graph format (e.g. tsv edge list vs graphml) matches the runner.

## Evidence

- [abstract] Weight function follows MetFusion-style formulation combining alpha-weighted MetFrag score average with sigmoid of spectral cosine times chemical similarity: "we have used MetFusion Scoring function to create a weight function, producing a weighted graph"
- [abstract] Random walk consumes the weighted graph to compute candidate probabilities from seed nodes: "This graph was subsequently used by the random walk to calculate the probability of 'walking' through a set of candidates, departing from seed nodes (represented by spectral library matches)"
- [methods] Fusion concept is used for ranking improvement from spectral library: "ChemWalker uses the Fusion concept, proposed on MetFusion, for ranking improvement from spectral library."
- [abstract] Construction combines spectral network and structural similarity among candidate structures: "Departing from the combination of a spectral network and the structural similarity among candidate structures, we have used MetFusion Scoring function to create a weight function, producing a"
- [intro] ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by in silico fragmentation, using random walk: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation"
- [readme] pyrwr handles weighted graphs and requires positive weights for RWR: [evidence span withheld — anchor status: external_doc]
- [methods] MetFrag CL jar used for in silico annotation runs feeding the weight function: "ChemWalker was tested on MetFrag2.3.-CL.jar"
