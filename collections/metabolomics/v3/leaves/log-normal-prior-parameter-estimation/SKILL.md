---
name: log-normal-prior-parameter-estimation
description: Use when building ZODIAC-style weighted assignment graphs from SIRIUS
  top-50 formula candidates and fragmentation trees, and you need data-driven prior
  parameters (mu, sigma^2, tau) for similarity-based Bayesian reranking.
when_to_use_negative:
- Analyzing a single compound in isolation with no related compounds in the dataset,
  since the similarity prior requires reciprocal plausibility edges between candidate
  formulas of multiple compounds
- Input lacks SIRIUS fragmentation trees or top-50 formula candidates (e.g., raw mzML/mzXML
  not yet processed) — the prior cannot be estimated without tree similarity scores
- The goal is de novo candidate generation rather than reranking an existing SIRIUS
  candidate list
tools:
- name: SIRIUS
  role: Computes the fragmentation trees from MS/MS spectra that are compared to produce
    the pairwise similarity scores sampled for prior estimation
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Consumes the estimated (mu, sigma^2, tau) parameters as the prior in its weighted
    assignment graph and Gibbs-sampling-based Bayesian reranking of formula candidates
  repo: https://bio.informatik.uni-jena.de/software/sirius/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:dacbf0c9c3692513b584c5c8a83fbb6e18d6e19aeb2eabcfe10a282429ef4446
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Log-normal prior parameter estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates ZODIAC's Bayesian prior on pairwise fragmentation-tree similarity scores by sampling 100,000 non-zero similarity scores and fitting a log-normal distribution (mu = median, sigma^2 via MAD) with a 95% quantile threshold tau. These parameters drive the node/edge weights of the molecular formula assignment graph used to rerank SIRIUS's top-50 formula candidates.

## When to use

Use when building ZODIAC-style weighted assignment graphs from SIRIUS top-50 formula candidates and fragmentation trees, and you need data-driven prior parameters (mu, sigma^2, tau) for similarity-based Bayesian reranking.

## When NOT to use

- Analyzing a single compound in isolation with no related compounds in the dataset, since the similarity prior requires reciprocal plausibility edges between candidate formulas of multiple compounds
- Input lacks SIRIUS fragmentation trees or top-50 formula candidates (e.g., raw mzML/mzXML not yet processed) — the prior cannot be estimated without tree similarity scores
- The goal is de novo candidate generation rather than reranking an existing SIRIUS candidate list

## Inputs

- SIRIUS top-50 molecular formula candidate results per compound (with candidate scores)
- SIRIUS fragmentation trees for the MS/MS spectra of the monoisotopic peaks
- Pairwise fragmentation-tree similarity scores s(u,v) between candidate formulas of related compounds in the LC-MS/MS dataset

## Outputs

- Log-normal prior parameters (mu, sigma^2) estimated from sampled similarity scores
- Similarity threshold tau (95% quantile of the 100,000 sampled non-zero scores)
- Weighted assignment graph (softmax node likelihoods, similarity-scored edges with anchor bonuses), exportable as JSON/TSV

## How to apply

First compute pairwise fragmentation-tree similarity scores s(u,v) between candidate formulas of related compounds from SIRIUS fragmentation trees, normalizing by weighted tree sizes with Pareto-based fragment importance weights. Sample 100,000 non-zero similarity scores from this distribution and estimate the log-normal prior: set mu to the median of the samples and sigma^2 via the median absolute deviation (MAD). Set the similarity threshold tau at the 95% quantile of the sampled non-zero score distribution, so that edges below tau contribute negligible prior support. These parameters feed the weighted assignment graph alongside softmax node likelihoods over SIRIUS candidate scores and the anchor bonus psi (min_l = 0.5, lambda = 1000) for spectral library hits, later consumed by ZODIAC's Gibbs sampler.

## Related tools

- **SIRIUS** (Computes the fragmentation trees from MS/MS spectra that are compared to produce the pairwise similarity scores sampled for prior estimation) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Consumes the estimated (mu, sigma^2, tau) parameters as the prior in its weighted assignment graph and Gibbs-sampling-based Bayesian reranking of formula candidates) — https://bio.informatik.uni-jena.de/software/sirius/

## Examples

```
import numpy as np; mu = np.median(s); sigma2 = (np.median(np.abs(s - mu)) / 0.6745)**2; tau = np.quantile(s, 0.95)
```

## Evaluation signals

- tau must equal the 95% quantile of the 100,000 sampled non-zero similarity scores
- mu must equal the median of the sampled non-zero scores; sigma^2 must be derived from the median absolute deviation (MAD) of the same samples
- Only non-zero similarity scores may enter the sampling; zero scores are excluded before fitting
- The exported graph must contain softmax node likelihoods normalized per compound (one node per candidate formula plus a dummy node per compound)
- Anchor-bonus parameters must match psi with min_l = 0.5 and lambda = 1000 (bonus reduced by 0.1 for analogue-mode hits)

## Limitations

- Library-hit anchors have only a small effect on molecular formula annotations, so the anchor bonus contributes little to the final posterior
- High-quality adduct annotations should be established during preprocessing, as they cannot be reliably deduced from isotope pattern and MS/MS data
- The prior is fitted per-dataset by sampling; it assumes the non-zero similarity score distribution is well approximated as log-normal and requires sufficient related compounds to sample from

## Evidence

- [abstract] abstract: [evidence span withheld — anchor status: not_found]
