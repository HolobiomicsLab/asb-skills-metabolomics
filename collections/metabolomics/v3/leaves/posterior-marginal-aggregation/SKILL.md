---
name: posterior-marginal-aggregation
description: Use when ZODIAC Gibbs sampling (ten parallel chains, 1,000 burn-in +
  2,000 sampling epochs) has run on a compound–candidate formula similarity graph
  and you need per-compound posterior probabilities for every SIRIUS formula candidate.
when_to_use_negative:
- You only need SIRIUS's own top-1 formula annotation and are not doing network-based
  reranking of the top 50 candidates
- Compounds are isolated in the ZODIAC similarity network (no pairwise-score edges),
  so Gibbs marginals would largely mirror the raw SIRIUS candidate scores
- You are analyzing raw mzML/mzXML spectra or computing fragmentation trees — this
  skill operates downstream, on ZODIAC's Gibbs samples, not on spectra
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: ZODIAC
  role: Produces the Gibbs sampling states (ten parallel chains, burn-in 1,000 epochs,
    sampling 2,000 epochs) whose thinned samples are aggregated into posterior marginals
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Supplies the candidate set — the top 50 molecular formula annotations per
    compound — over which the posterior marginals are computed
  repo: https://github.com/sirius-ms/sirius
- name: PySirius
  role: Optional Python SDK for the SIRIUS/OpenAPI API used to drive SIRIUS/ZODIAC
    jobs and retrieve results programmatically
  repo: https://github.com/sirius-ms/sirius-client-openAPI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:7603937dc00c925a8790b95def0ec22de797a2da97081196dd28dab5fb99c456
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
  - PySirius
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# posterior-marginal-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates thinned Gibbs-sampler states from ZODIAC's parallel Markov chains into per-compound posterior marginals over SIRIUS molecular formula candidates, so reranked formula probabilities (not just a top-1 pick) can be inspected, thresholded, or propagated downstream.

## When to use

Use when ZODIAC Gibbs sampling (ten parallel chains, 1,000 burn-in + 2,000 sampling epochs) has run on a compound–candidate formula similarity graph and you need per-compound posterior probabilities for every SIRIUS formula candidate.

## When NOT to use

- You only need SIRIUS's own top-1 formula annotation and are not doing network-based reranking of the top 50 candidates
- Compounds are isolated in the ZODIAC similarity network (no pairwise-score edges), so Gibbs marginals would largely mirror the raw SIRIUS candidate scores
- You are analyzing raw mzML/mzXML spectra or computing fragmentation trees — this skill operates downstream, on ZODIAC's Gibbs samples, not on spectra

## Inputs

- Thinned post-burn-in Gibbs chain states (compound → formula assignment per retained epoch, every 10th epoch)
- Compound × candidate-formula set from the top 50 SIRIUS molecular formula annotations
- Weighted dendroides-style similarity graph (compound × formula nodes, ZODIAC pairwise-score edges)
- Per-epoch trajectory table (epoch, total score, identification rate)

## Outputs

- Posterior marginal matrix compounds × formula candidates with values in [0,1] (e.g. zodiac_dendroides_marginals.csv)
- Sampling trajectory table (epoch, total score, identification rate, e.g. zodiac_dendroides_trajectory.csv)

## How to apply

Run ZODIAC's Gibbs sampler on the weighted compound × formula-candidate graph (e.g. the dendroides SIRIUS-pairwise-score network) with ten parallel chains, a 1,000-epoch burn-in and 2,000 sampling epochs; thin by retaining every 10th post-burn-in state. Concatenate the thinned states across the ten chains and, for each compound, compute the empirical frequency of each molecular formula candidate across all retained states — this frequency is the posterior marginal and must lie in [0,1]. Log the per-epoch total score and identification rate into a trajectory table so you can verify the chains plateaued after burn-in before trusting the marginals. Rationale: ZODIAC reranks the top 50 SIRIUS candidates with Bayesian statistics via Gibbs sampling, and aggregating across parallel chains and thinned states makes the sampling more robust while enabling parallelization.

## Related tools

- **ZODIAC** (Produces the Gibbs sampling states (ten parallel chains, burn-in 1,000 epochs, sampling 2,000 epochs) whose thinned samples are aggregated into posterior marginals) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Supplies the candidate set — the top 50 molecular formula annotations per compound — over which the posterior marginals are computed) — https://github.com/sirius-ms/sirius
- **PySirius** (Optional Python SDK for the SIRIUS/OpenAPI API used to drive SIRIUS/ZODIAC jobs and retrieve results programmatically) — https://github.com/sirius-ms/sirius-client-openAPI

## Examples

```
python -c "import pandas as pd; s=pd.read_csv('zodiac_dendroides_samples.csv'); m=s.groupby(['compound','formula'])['state'].mean().reset_index(); m.to_csv('zodiac_dendroides_marginals.csv', index=False)"
```

## Evaluation signals

- Every entry of the aggregated marginal matrix is within [0,1], and for each compound the marginals over its top-50 candidates sum to 1
- Trajectory table shows the total score stabilizing (plateau) during the 1,000-epoch burn-in before the 2,000 sampled epochs
- Identification rates are consistent across the ten parallel chains after burn-in; divergent chains indicate insufficient mixing
- Aggregation used only thinned states (every 10th epoch) across all ten chains — recompute with a different thinning/seed and marginals should change only marginally
- Marginals on compounds with a spectral-library-hit anchor should not shift dramatically, since anchors were reported to have only a small effect

## Limitations

- The 10-chain / 1,000 burn-in / 2,000 sampling-epoch configuration was chosen empirically for parallelization and robustness, not derived from a convergence diagnostic
- The optimized (fast conditional update, ~25-fold) Gibbs sampler was not evaluated against a naive implementation; the speedup is only a theoretical estimate
- Library-hit anchors contribute only a small effect to the resulting formula annotations, so they should not be relied on to sharpen marginals

## Evidence

- [abstract] ZODIAC reranks SIRIUS' molecular formula candidates, combining fragmentation tree computation with Bayesian statistics using Gibbs sampling.: [evidence span withheld — anchor status: not_found]
