---
name: assignment-graph-construction
description: Use when you have SIRIUS top-50 molecular formula candidate lists for
  a batch of compounds (e.g. from a dendroides-style LC-MS/MS run) and need to build
  the weighted candidate/edge graph that ZODIAC's Bayesian Gibbs-sampling model consumes.
when_to_use_negative:
- If you only have a single compound's SIRIUS result with no related-compound network
  context — the graph construction depends on pairwise fragmentation-tree similarity
  across related compounds and has no edges to form.
- If your goal is adduct or structural annotation rather than molecular formula reranking;
  the graph is specifically the candidate formula node/edge structure consumed by
  ZODIAC's Gibbs sampler.
- If the SIRIUS fragmentation trees have not yet been computed for the candidate formulas
  — the similarity scores (equations 6-7) cannot be calculated.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Provides the top-50 molecular formula candidates and the fragmentation trees
    used to compute pairwise similarity scores for edges of the assignment graph
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Consumes the constructed weighted assignment graph (nodes with softmax likelihoods,
    edges with similarity scores and anchor bonuses) for Bayesian reranking via Gibbs
    sampling
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:eb0b9e05c861d1211f9f8fefff59c6cfe1147f0f7f451a24603b665c1dece705
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# assignment-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs ZODIAC's weighted molecular-formula assignment graph, where nodes are SIRIUS top-50 formula candidates (plus a dummy node) weighted by softmax likelihoods, and edges encode fragmentation-tree similarity scores with spectral-library anchor bonuses. This graph is the input that ZODIAC's Gibbs sampler operates on to rerank molecular formulas in LC-MS/MS datasets.

## When to use

Use when you have SIRIUS top-50 molecular formula candidate lists for a batch of compounds (e.g. from a dendroides-style LC-MS/MS run) and need to build the weighted candidate/edge graph that ZODIAC's Bayesian Gibbs-sampling model consumes.

## When NOT to use

- If you only have a single compound's SIRIUS result with no related-compound network context — the graph construction depends on pairwise fragmentation-tree similarity across related compounds and has no edges to form.
- If your goal is adduct or structural annotation rather than molecular formula reranking; the graph is specifically the candidate formula node/edge structure consumed by ZODIAC's Gibbs sampler.
- If the SIRIUS fragmentation trees have not yet been computed for the candidate formulas — the similarity scores (equations 6-7) cannot be calculated.

## Inputs

- SIRIUS top-50 molecular formula candidate results (per compound) for an LC-MS/MS dataset
- SIRIUS fragmentation trees for the candidate formulas
- Spectral library search hits (library anchors, optionally analogue-mode hits)
- LC-MS/MS processed data (e.g. mzML/mzXML via OpenMS)

## Outputs

- Weighted assignment graph in JSON/TSV (nodes = formula candidates + dummy node, node weights = softmax likelihoods; edges = similarity scores + anchor bonuses)
- Estimated prior parameters (mu, sigma^2) and similarity threshold tau
- Per-compound node sets with normalized softmax weights

## How to apply

Instantiate one candidate node per molecular formula per compound plus a dummy node per compound, and assign each node a softmax likelihood derived from the SIRIUS candidate scores. Compute pairwise fragmentation-tree similarity scores s(u,v) between candidate formulas of related compounds using SIRIUS fragmentation trees, normalizing by weighted tree sizes (equations 6-7) with Pareto-based fragment importance weights. Sample 100,000 non-zero similarity scores to estimate the log-normal p-value prior: mu = median, sigma^2 via MAD, and set the edge threshold tau at the 95% quantile of that sampled distribution. Apply the anchor bonus psi to edges backed by spectral library hits (min_l = 0.5, lambda = 1000; reduce bonus by 0.1 in analogue mode). Export the assembled weighted assignment graph as JSON/TSV together with the estimated (mu, sigma^2, tau) parameters so the Gibbs sampler can consume them.

## Related tools

- **SIRIUS** (Provides the top-50 molecular formula candidates and the fragmentation trees used to compute pairwise similarity scores for edges of the assignment graph) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Consumes the constructed weighted assignment graph (nodes with softmax likelihoods, edges with similarity scores and anchor bonuses) for Bayesian reranking via Gibbs sampling) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --formula-annotation --zodiac --output zodiac_graph.json dendroides.mzML
```

## Evaluation signals

- Verify the exported graph contains exactly one dummy node per compound and that softmax node weights sum to 1 per compound.
- Check that tau equals the 95% quantile of the 100,000 sampled non-zero similarity scores.
- Confirm that all edges carry both a similarity score and, where applicable, an anchor bonus value; analogue-mode anchor edges should have the bonus reduced by 0.1 relative to direct library hits.
- Cross-check the number of candidate nodes per compound against the SIRIUS top-50 output (should match the top-50 list length for each compound).

## Limitations

- The method relies on pairwise fragmentation-tree similarity across related compounds; if the LC-MS/MS dataset lacks sufficient structural or spectral relationships, edges are sparse and the graph under-constrains the Gibbs sampler.
- SIRIUS's performance drops substantially for compounds above 500 Da, which limits the quality of the candidate nodes feeding the graph.
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data; high-quality adduct annotations must be established during preprocessing or the graph's candidate sets may be wrong.
- Library-hit anchors contribute only a small effect to the final molecular formula annotations.

## Evidence

- [results] ZODIAC builds upon SIRIUS and uses, say, the top 50 molecular formula annotations from SIRIUS as candidates for one compound: "ZODIAC builds upon SIRIUS and uses, say, the top 50 molecular formula annotations from SIRIUS as candidates for one compound"
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small molecules of biological interest".: [evidence span withheld — anchor status: external_doc]
