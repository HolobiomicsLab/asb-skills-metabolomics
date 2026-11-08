---
name: fragmentation-tree-similarity-scoring
description: Use when reranking SIRIUS top-50 molecular formula candidates across
  an LC-MS/MS dataset where related compounds should share similar formulas, i.e.
  building ZODIAC's Bayesian assignment graph.
when_to_use_negative:
- When only a single isolated compound is analyzed with no related compounds in the
  LC-MS/MS dataset — tree-similarity edges require compounds that plausibly share
  formulas
- When input is raw mzML/mzXML that has not yet been processed into SIRIUS top-50
  candidates and fragmentation trees (preprocess with OpenMS and SIRIUS first)
- When a spectral library search already yields confident identifications — library-hit
  anchors have only a small effect on molecular formula annotations
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: SIRIUS
  role: Computes the fragmentation trees and top-50 molecular formula candidates whose
    pairwise similarity drives the graph edges and node likelihoods
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Implements the weighted assignment graph, log-normal prior estimation, anchor
    bonus psi, and Gibbs sampling reranking built on the similarity scores
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS 2.4.0 (modified)
  role: Preprocessing of mzML/mzXML input and discarding low-quality MS/MS spectra
    before formula annotation; a modified build plus patch file is required for the
    ZODIAC workflow
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:54d1e5dd7c24c7d981c7b8e1af0d7669af9f8e32df5582a1fa9648801d0f04cb
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
  - OpenMS 2.4.0 (modified)
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fragmentation-tree-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise fragmentation-tree similarity scores s(u,v) between SIRIUS molecular formula candidates of related compounds and converts them into log-normal priors plus anchor bonuses for ZODIAC's weighted assignment graph.

## When to use

Use when reranking SIRIUS top-50 molecular formula candidates across an LC-MS/MS dataset where related compounds should share similar formulas, i.e. building ZODIAC's Bayesian assignment graph.

## When NOT to use

- When only a single isolated compound is analyzed with no related compounds in the LC-MS/MS dataset — tree-similarity edges require compounds that plausibly share formulas
- When input is raw mzML/mzXML that has not yet been processed into SIRIUS top-50 candidates and fragmentation trees (preprocess with OpenMS and SIRIUS first)
- When a spectral library search already yields confident identifications — library-hit anchors have only a small effect on molecular formula annotations

## Inputs

- SIRIUS top-50 molecular formula candidate results per compound (e.g. from the dendroides dataset)
- SIRIUS fragmentation trees for the MS/MS spectrum of each monoisotopic peak
- Spectral library search hits (anchors), including analogue-mode hits
- Per-compound candidate scores used for softmax node likelihoods

## Outputs

- Weighted assignment graph (nodes with softmax likelihoods, edges with similarity scores and anchor bonuses) exported as JSON/TSV
- Estimated log-normal prior parameters (mu, sigma^2) from 100,000 sampled non-zero similarity scores
- Similarity threshold tau at the 95% quantile of the sampled non-zero score distribution

## How to apply

First run SIRIUS per compound to obtain the top-50 molecular formula candidates and their fragmentation trees, then compute pairwise tree similarity s(u,v) between candidate formulas of related compounds, normalizing by weighted tree sizes (equations 6-7) with Pareto-based fragment importance weights. Sample 100,000 non-zero similarity scores and estimate the log-normal p-value prior with mu as the median and sigma^2 via the median absolute deviation (MAD); set the similarity threshold tau at the 95% quantile of the sampled non-zero score distribution. For edges backed by spectral library hits (anchors), apply bonus psi with min_l = 0.5 and lambda = 1000, reducing the bonus by 0.1 for analogue-mode hits. Assemble the weighted assignment graph — one node per candidate formula per compound plus a dummy node carrying a softmax likelihood over candidate scores — and export it as JSON/TSV together with the estimated (mu, sigma^2, tau) parameters. The rationale is that ZODIAC's Gibbs sampling exploits reciprocal plausibility: compounds in the same LC-MS/MS sample whose fragmentation trees are similar should be assigned similar or identical molecular formulas.

## Related tools

- **SIRIUS** (Computes the fragmentation trees and top-50 molecular formula candidates whose pairwise similarity drives the graph edges and node likelihoods) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Implements the weighted assignment graph, log-normal prior estimation, anchor bonus psi, and Gibbs sampling reranking built on the similarity scores) — https://github.com/sirius-ms/sirius
- **OpenMS 2.4.0 (modified)** (Preprocessing of mzML/mzXML input and discarding low-quality MS/MS spectra before formula annotation; a modified build plus patch file is required for the ZODIAC workflow) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- Exported graph contains exactly one dummy node per compound
- Softmax node weights are normalized to sum to 1 per compound
- tau equals the 95% quantile of the 100,000 sampled non-zero similarity scores
- Edges backed by spectral library hits carry anchor bonus psi with min_l = 0.5 and lambda = 1000, and analogue-mode hits receive the bonus reduced by 0.1
- Prior parameters (mu, sigma^2) reproduce the median and MAD of the sampled non-zero score distribution

## Limitations

- Library-hit anchors contribute only a small effect to molecular formula annotations, so the anchor bonus alone will not rescue weak candidates
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data; high-quality adduct annotations must be established during preprocessing
- SIRIUS performance drops substantially for compounds above 500 Da, which is the regime ZODIAC's tree-similarity priors are designed to improve
- Reproducibility requires a modified version of OpenMS 2.4.0 plus a separate patch file, rather than stock OpenMS

## Evidence

- [abstract] abstract: [evidence span withheld — anchor status: not_found]
- [discussion] discussion: [evidence span withheld — anchor status: not_found]
