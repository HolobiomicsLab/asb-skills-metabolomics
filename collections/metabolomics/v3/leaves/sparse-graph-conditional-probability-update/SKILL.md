---
name: sparse-graph-conditional-probability-update
description: Use when Gibbs sampling must be run over a large weighted similarity
  graph (e.g., compounds × candidate molecular formulas with pairwise ZODIAC scores)
  and a full conditional recomputation over all nodes is too slow; the sparse conditional
  update touches only flipped-node neighborhoods.
when_to_use_negative:
- The similarity graph is dense — the O(|V(c)| + deg(u) + deg(v)) conditional update
  degenerates toward a full-graph recomputation and the sparsity-based speedup vanishes.
- No pairwise scores exist between compounds' candidate formulas (e.g., only isolated
  per-compound SIRIUS candidate lists), so the network-based conditional structure
  that Lemma 2 exploits is absent.
- An exact (non-sampled) posterior or MAP assignment is required — this skill implements
  stochastic Gibbs sampling, not deterministic optimization.
tools:
- name: ZODIAC
  role: Implements the Gibbs sampler with the fast (Lemma 2) conditional update; reranks
    SIRIUS molecular formula candidates using Bayesian statistics; shipped inside
    SIRIUS
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: SIRIUS
  role: Supplies the candidate molecular formula annotations (top 50 per compound)
    that initialize and constrain each Gibbs chain's compound variables
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:6db83b69ae498f73a26b229ca27d1a327f3c9dd763d57f9977190d3e777fca3a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sparse-graph-conditional-probability-update

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

ZODIAC's engineered Gibbs sampler updates each compound's molecular formula assignment using only the pairwise potentials incident to the flipped compound and its neighbors, computed in O(|V(c)| + deg(u) + deg(v)) time per conditional (Lemma 2). This exploits the sparsity of the formula-similarity network to make Gibbs sampling very swift in practice.

## When to use

Use when Gibbs sampling must be run over a large weighted similarity graph (e.g., compounds × candidate molecular formulas with pairwise ZODIAC scores) and a full conditional recomputation over all nodes is too slow; the sparse conditional update touches only flipped-node neighborhoods.

## When NOT to use

- The similarity graph is dense — the O(|V(c)| + deg(u) + deg(v)) conditional update degenerates toward a full-graph recomputation and the sparsity-based speedup vanishes.
- No pairwise scores exist between compounds' candidate formulas (e.g., only isolated per-compound SIRIUS candidate lists), so the network-based conditional structure that Lemma 2 exploits is absent.
- An exact (non-sampled) posterior or MAP assignment is required — this skill implements stochastic Gibbs sampling, not deterministic optimization.

## Inputs

- Weighted formula-similarity graph (compound × candidate-formula nodes, ZODIAC pairwise-score edges), e.g., the dendroides graph
- SIRIUS candidate molecular formula scores per compound (e.g., top 50 candidates per compound from SIRIUS 4)

## Outputs

- zodiac_dendroides_marginals.csv — per-compound posterior marginals over molecular formula candidates (compounds × candidates, all values in [0,1])
- zodiac_dendroides_trajectory.csv — per-epoch table of total score and identification rate for the chains

## How to apply

Build the weighted formula-similarity graph from ZODIAC pairwise scores, with compound nodes linked to their SIRIUS candidate molecular formulas (e.g., the top 50 SIRIUS annotations per compound). Initialize ZODIAC's ten parallel Gibbs chains, drawing each compound's molecular formula from its SIRIUS candidate scores. Each epoch, sweep compounds and resample each formula from its conditional computed in O(|V(c)| + deg(u) + deg(v)) time per Lemma 2, touching only the pairwise potentials incident to the flipped compound and its neighbors instead of recomputing over the whole graph; run a 1,000-epoch burn-in followed by 2,000 sampled epochs, retaining every 10th state. After every epoch, record the chains' total score and the identification rate into a trajectory table. Aggregate the thinned samples across the ten chains into per-compound posterior marginals over molecular formula candidates. The rationale is algorithm engineering for sparsity: the article reports an estimated ~25-fold speedup over a naive full-conditional implementation, driven by degree-bounded neighborhoods rather than dense graph traversal.

## Related tools

- **ZODIAC** (Implements the Gibbs sampler with the fast (Lemma 2) conditional update; reranks SIRIUS molecular formula candidates using Bayesian statistics; shipped inside SIRIUS) — https://bio.informatik.uni-jena.de/software/sirius/
- **SIRIUS** (Supplies the candidate molecular formula annotations (top 50 per compound) that initialize and constrain each Gibbs chain's compound variables) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Trajectory table records a total score and identification rate for every epoch across the 1,000 burn-in and 2,000 sampling epochs; scores should stabilize after burn-in.
- Posterior marginals in the compounds × candidates CSV are all within [0,1] and each compound's marginal row is normalized to the candidate set.
- Thinning discipline is respected: only every 10th sampled state (after the 1,000-epoch burn-in) contributes to the aggregated marginals across the ten parallel chains.
- Per-epoch update cost scales with node neighborhood size (|V(c)| + deg(u) + deg(v)), not total graph size; wall-clock behavior should be consistent with the article's ~25-fold estimated speedup over a naive sampler.
- Identification rate computed against ground truth established by spectral library search and manual validation should meet or exceed the SIRIUS-only baseline (e.g., the 16.2-fold reduction in incorrect annotations reported for the challenging dendroides plant extract dataset).

## Limitations

- The ~25-fold speedup of the optimized Gibbs sampler was estimated from theoretical considerations only — it was not evaluated against a naïve implementation in the paper.
- The sampler is stochastic; the ten parallel chains, 1,000-epoch burn-in, and 2,000-epoch sampling setup were chosen for parallelization and robustness, so results depend on sampling variance and thinning.
- Spectral library-hit anchors contribute only a small effect to molecular formula annotations, so the network-driven conditional update is the dominant signal.

## Evidence

- [results] We did not evaluate our optimized Gibbs sampler against a na¨ıve version: [evidence span withheld — anchor status: not_found]
