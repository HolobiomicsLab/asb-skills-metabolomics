---
name: mcmc-convergence-diagnostics
description: Use when running a Gibbs sampler over a candidate-assignment network
  (e.g., ZODIAC reranking SIRIUS formula candidates) and you must set chain count,
  burn-in, sampling epochs, and verify the chains have stabilized before trusting
  posterior marginals.
when_to_use_negative:
- The inference problem is not a Gibbs-samplable Markov network over discrete candidate
  assignments (e.g., no pairwise potentials between compound candidates), since the
  conditional-update and multi-chain design assume that structure
- You need a formally validated convergence measure or an empirically benchmarked
  sampler speedup — the paper only estimates the ~25-fold speedup theoretically and
  uses trajectory inspection, not formal diagnostics
- You need deterministic, reproducible reranking of SIRIUS candidates without stochastic
  sampling — a single-pass scoring approach avoids chain/burn-in configuration entirely
tools:
- name: SIRIUS
  role: Hosts the ZODIAC Gibbs sampler; supplies the molecular formula candidates
    (top 50 annotations) whose reranking the chains monitor for convergence
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS
  role: Modified OpenMS 2.4.0 processes input mzML/mzXML files and discards low-quality
    MS/MS spectra upstream of the ZODIAC sampling workflow
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:5cfca6e3df43e4ff305374391392b3ae5cc7ee6a007e945a368c54a6460cd02f
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mcmc-convergence-diagnostics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Diagnose convergence and stability of Gibbs-sampler-based inference (as instantiated in ZODIAC for molecular formula reranking) by monitoring parallel chains, burn-in vs. sampling epochs, and per-epoch trajectory statistics such as total score and identification rate.

## When to use

Use when running a Gibbs sampler over a candidate-assignment network (e.g., ZODIAC reranking SIRIUS formula candidates) and you must set chain count, burn-in, sampling epochs, and verify the chains have stabilized before trusting posterior marginals.

## When NOT to use

- The inference problem is not a Gibbs-samplable Markov network over discrete candidate assignments (e.g., no pairwise potentials between compound candidates), since the conditional-update and multi-chain design assume that structure
- You need a formally validated convergence measure or an empirically benchmarked sampler speedup — the paper only estimates the ~25-fold speedup theoretically and uses trajectory inspection, not formal diagnostics
- You need deterministic, reproducible reranking of SIRIUS candidates without stochastic sampling — a single-pass scoring approach avoids chain/burn-in configuration entirely

## Inputs

- Weighted compound × candidate-formula similarity graph (e.g., dendroides formula-similarity graph with ZODIAC pairwise-score edges)
- SIRIUS molecular formula candidate scores per compound (top 50 candidates per compound)
- Sampler run parameters: number of chains (10), burn-in epochs (1,000), sampling epochs (2,000), thinning interval (every 10th state)

## Outputs

- Trajectory table (epoch, total score, identification rate) recorded after every epoch, e.g., zodiac_dendroides_trajectory.csv
- Per-compound posterior marginals over molecular formula candidates aggregated across all 10 chains, e.g., zodiac_dendroides_marginals.csv with all values in [0,1]

## How to apply

Configure the sampler as ZODIAC does: initialize ten parallel Markov chains, drawing each compound's molecular formula from its SIRIUS candidate scores (top 50 SIRIUS annotations are used as candidates). Run a burn-in of 1,000 epochs, then sample for 2,000 epochs; the multi-chain design was chosen to allow parallelization (40 cores) and to make sampling more robust. After every epoch, record the chains' total score and the identification rate into a trajectory table and inspect these curves for plateauing as the convergence signal; retain every 10th sampled state (thinning) when aggregating across the ten chains into per-compound posterior marginals. Rely on algorithm engineering for speed — the fast conditional update (Lemma 2) computes each conditional in O(|V(c)| + deg(u) + deg(v)) time by touching only potentials incident to the flipped compound and its neighbors. Note the paper provides no formal diagnostic (e.g., R-hat); convergence assessment here is trajectory-based (score/identification-rate stability across chains), so report it as such rather than claiming formal proof of mixing.

## Related tools

- **SIRIUS** (Hosts the ZODIAC Gibbs sampler; supplies the molecular formula candidates (top 50 annotations) whose reranking the chains monitor for convergence) — https://github.com/sirius-ms/sirius
- **OpenMS** (Modified OpenMS 2.4.0 processes input mzML/mzXML files and discards low-quality MS/MS spectra upstream of the ZODIAC sampling workflow) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- Trajectory table shows total score and identification rate reaching a stable plateau during the 1,000 burn-in epochs and remaining stable across the 2,000 sampled epochs
- All ten parallel chains yield consistent per-compound posterior marginals after aggregation; no single chain diverges from the consensus
- Every value in the marginals output (zodiac_dendroides_marginals.csv) lies in [0,1] and each compound's marginals cover its SIRIUS candidate set
- Identification rate in the trajectory is consistent with ZODIAC's reported improvement over SIRIUS (e.g., the 16.2-fold decrease in incorrect annotations on the challenging dendroides plant extract)
- Runtime is consistent with the engineered sampler (est. ~25-fold speedup from Lemma 2 conditional updates); a naive per-epoch full recompute would be markedly slower

## Limitations

- The optimized Gibbs sampler was not evaluated against a naive version; the ~25-fold speedup is estimated only from theoretical considerations
- No formal MCMC convergence diagnostic (e.g., R-hat, effective sample size) is reported — convergence is judged from the recorded total-score/identification-rate trajectory across chains
- Ten parallel chains with 1,000 burn-in / 2,000 sampling epochs is a practical configuration for robustness and parallelization, not a validated guarantee of mixing for arbitrary graphs

## Evidence
