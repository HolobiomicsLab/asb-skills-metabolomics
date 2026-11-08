---
name: softmax-candidate-likelihood-modeling
description: Use when you have SIRIUS top-50 formula candidates for multiple related
  compounds in an LC-MS/MS dataset and need to build the node/edge graph (softmax
  likelihoods, tree-similarity edges, anchor bonuses) that ZODIAC's Bayesian model
  requires.
when_to_use_negative:
- Isolated single-compound formula annotation with no related compounds in the dataset
  — ZODIAC's prior derives from reciprocal plausibility (tree similarity) within an
  LC-MS/MS dataset, so there are no informative edges
- Precursor data lacking high-quality adduct annotations established during preprocessing,
  since adduct information cannot be reliably deduced from isotope pattern and MS/MS
  data alone
- Workflows locked to stock OpenMS — reproducibility requires the modified OpenMS
  2.4.0 plus a separate patch file from bio.informatik.uni-jena.de/data/
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Produces the top-50 molecular formula candidate scores and the fragmentation
    trees from which pairwise tree-similarity scores and softmax node likelihoods
    are derived
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Implements the weighted assignment graph construction (softmax nodes, similarity
    edges, anchor bonuses) feeding the Gibbs-sampling Bayesian reranker
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS
  role: Preprocesses input mzML/mzXML files and discards low-quality MS/MS spectra
    before candidate and tree computation (requires the modified 2.4.0 build)
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:9452c0f27a845eaa7d594167d3bfe3d876731aefa807d4064e5ee5700a414de1
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
  - OpenMS
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# softmax-candidate-likelihood-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs ZODIAC's weighted assignment graph by assigning each SIRIUS top-50 molecular formula candidate (plus a dummy node) a softmax likelihood over candidate scores, and weighting edges with fragmentation-tree similarity scores and spectral-library anchor bonuses for downstream Gibbs-sampling reranking.

## When to use

Use when you have SIRIUS top-50 formula candidates for multiple related compounds in an LC-MS/MS dataset and need to build the node/edge graph (softmax likelihoods, tree-similarity edges, anchor bonuses) that ZODIAC's Bayesian model requires.

## When NOT to use

- Isolated single-compound formula annotation with no related compounds in the dataset — ZODIAC's prior derives from reciprocal plausibility (tree similarity) within an LC-MS/MS dataset, so there are no informative edges
- Precursor data lacking high-quality adduct annotations established during preprocessing, since adduct information cannot be reliably deduced from isotope pattern and MS/MS data alone
- Workflows locked to stock OpenMS — reproducibility requires the modified OpenMS 2.4.0 plus a separate patch file from bio.informatik.uni-jena.de/data/

## Inputs

- SIRIUS top-50 molecular formula candidate scores per compound (deposited candidate results)
- SIRIUS fragmentation trees for the MS/MS spectrum of each monoisotopic peak
- Spectral library search hits (e.g. NIST17) including analogue-mode hits, used as anchors
- Preprocessed LC-MS/MS feature data (mzML/mzXML processed with modified OpenMS 2.4.0) with high-quality adduct annotations

## Outputs

- Weighted assignment graph: candidate nodes plus dummy nodes with softmax likelihoods, edges with fragmentation-tree similarity scores s(u,v) and anchor bonuses psi
- Estimated prior parameters (mu, sigma^2, tau) for the log-normal p-value model
- Graph exported as JSON/TSV

## How to apply

Load the deposited SIRIUS top-50 molecular formula candidate results (e.g. for the dendroides dataset) and instantiate one candidate node per formula per compound plus a dummy node, assigning each node a softmax likelihood over the candidate scores so weights normalize per compound. Compute pairwise fragmentation-tree similarity s(u,v) between candidate formulas of related compounds from SIRIUS fragmentation trees, normalizing by weighted tree sizes (equations 6-7) with Pareto-based fragment importance weights. Sample 100,000 non-zero similarity scores to estimate the log-normal p-value prior — mu as the median and sigma^2 via the median absolute deviation (MAD) — and set the similarity threshold tau at the 95% quantile of the sampled score distribution. For edges backed by spectral library hits (e.g. NIST17), apply the anchor bonus psi using min_l = 0.5 and lambda = 1000, reduced by 0.1 for analogue-mode hits. Finally, assemble the weighted assignment graph (softmax likelihoods on nodes, similarity scores and anchor bonuses on edges) and export it as JSON/TSV together with the estimated (mu, sigma^2, tau) parameters for Gibbs sampling.

## Related tools

- **SIRIUS** (Produces the top-50 molecular formula candidate scores and the fragmentation trees from which pairwise tree-similarity scores and softmax node likelihoods are derived) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Implements the weighted assignment graph construction (softmax nodes, similarity edges, anchor bonuses) feeding the Gibbs-sampling Bayesian reranker) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS** (Preprocesses input mzML/mzXML files and discards low-quality MS/MS spectra before candidate and tree computation (requires the modified 2.4.0 build)) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
sirius -i dendroides.mzML --formula --zodiac
```

## Evaluation signals

- Exported graph contains exactly one dummy node per compound in addition to the SIRIUS candidate nodes
- Softmax node weights are normalized to sum to 1 within each compound's candidate set
- tau equals the 95% quantile of the 100,000 sampled non-zero similarity scores; mu and sigma^2 match the median and MAD of those samples
- Edges backed by spectral library hits carry the anchor bonus psi with min_l = 0.5 and lambda = 1000, and analogue-mode hits show a 0.1 reduction of the bonus
- Serialized graph is valid JSON/TSV and includes (mu, sigma^2, tau) alongside node and edge weights

## Limitations

- Library-hit anchors contribute only a small effect to molecular formula annotations, so the anchor bonus should not be expected to dominate the reranking
- SIRIUS candidate quality degrades for large molecules — performance drops substantially for compounds above 500 Da, which limits the graph's candidate pools for high-mass metabolites
- No distinct difference was found between the cosine score of ZODIAC's correct and incorrect assignments, so cosine score cannot serve as an external quality check on the assembled graph

## Evidence

- [discussion] Requirement for preprocessing-time adduct annotations: [evidence span withheld — anchor status: not_found]
- [results] Input preprocessing with OpenMS: [evidence span withheld — anchor status: not_found]
