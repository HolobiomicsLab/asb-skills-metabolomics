---
name: molecular-formula-gibbs-sampling
description: Use when you have SIRIUS formula candidate scores (top ~50 per compound)
  for a whole LC-MS/MS sample and need de novo, database-independent formula reranking,
  especially for high-mass compounds (>500 Da).
when_to_use_negative:
- Input lacks SIRIUS fragmentation tree candidates — ZODIAC builds upon SIRIUS and
  reranks its top candidates, so it cannot run de novo from raw spectra alone.
- Analyzing isolated single-compound spectra with no sample context — the network-based
  model's power comes from pairwise similarities across compounds in a dataset.
- 'Adduct information is needed but unreliable: deduce high-quality adduct annotations
  during preprocessing, since adducts cannot be reliably inferred from isotope pattern
  and MS/MS data.'
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS
  role: Provides the top ~50 molecular formula annotations per compound (via isotope
    pattern analysis and fragmentation tree computation) that ZODIAC reranks; ZODIAC
    is implemented inside SIRIUS
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Network-based Gibbs sampler that reranks SIRIUS formula candidates using Bayesian
    statistics over pairwise potentials; run with ten parallel chains, 1,000 burn-in
    epochs, 2,000 sampling epochs
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS 2.4.0 (modified)
  role: Preprocesses input mzML/mzXML files and discards low-quality MS/MS spectra
    before ZODIAC; a modified release plus patch file is required for reproducibility
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:2ac9921aafac2c37f40b9ad739a6fabf9fc96ec6ba6d920262e3131ef076d746
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-formula-gibbs-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rerank SIRIUS molecular formula candidates using ZODIAC's network-based Gibbs sampler, which combines fragmentation tree scores with Bayesian statistics over pairwise similarity potentials across compounds in an LC-MS/MS sample. This addresses SIRIUS 4's performance drop for compounds above 500 Da.

## When to use

Use when you have SIRIUS formula candidate scores (top ~50 per compound) for a whole LC-MS/MS sample and need de novo, database-independent formula reranking, especially for high-mass compounds (>500 Da).

## When NOT to use

- Input lacks SIRIUS fragmentation tree candidates — ZODIAC builds upon SIRIUS and reranks its top candidates, so it cannot run de novo from raw spectra alone.
- Analyzing isolated single-compound spectra with no sample context — the network-based model's power comes from pairwise similarities across compounds in a dataset.
- Adduct information is needed but unreliable: deduce high-quality adduct annotations during preprocessing, since adducts cannot be reliably inferred from isotope pattern and MS/MS data.

## Inputs

- SIRIUS output: top 50 molecular formula candidates per compound with fragmentation tree scores (dendroides formula-similarity graph, compound × candidate-formula nodes)
- ZODIAC pairwise similarity scores between candidate formulas across compounds
- Optional: spectral library search hits to integrate as anchors
- Preprocessed mzML/mzXML files processed with OpenMS (low-quality MS/MS spectra discarded)

## Outputs

- Per-compound posterior marginals for every molecular formula candidate (compounds × candidates matrix, all values in [0,1])
- Gibbs sampling trajectory table (epoch, total score, identification rate)
- Reranked molecular formula assignments per compound

## How to apply

Build a weighted formula-similarity graph whose nodes are compound × candidate-formula pairs and whose edges carry ZODIAC pairwise scores computed from SIRIUS fragmentation tree information and isotope patterns. Initialize ten parallel Markov chains, drawing each compound's formula from its SIRIUS candidate scores, then run a burn-in of 1,000 epochs followed by 2,000 sampled epochs; each epoch sweeps compounds and resamples each formula from its conditional in O(|V(c)| + deg(u) + deg(v)) time per Lemma 2, touching only pairwise potentials incident to the flipped compound and its neighbors. Thinning (e.g. retaining every 10th state) yields per-compound posterior marginals over all candidates; spectral library search hits can be integrated as anchors in the stochastic model. Rationale: Gibbs sampling exploits correlations between compounds sharing related formulas, which is what drives the 16.2-fold decrease in incorrect annotations on the challenging dendroides plant extract dataset (most compounds above 700 Dalton).

## Related tools

- **SIRIUS** (Provides the top ~50 molecular formula annotations per compound (via isotope pattern analysis and fragmentation tree computation) that ZODIAC reranks; ZODIAC is implemented inside SIRIUS) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Network-based Gibbs sampler that reranks SIRIUS formula candidates using Bayesian statistics over pairwise potentials; run with ten parallel chains, 1,000 burn-in epochs, 2,000 sampling epochs) — https://github.com/sirius-ms/sirius
- **OpenMS 2.4.0 (modified)** (Preprocesses input mzML/mzXML files and discards low-quality MS/MS spectra before ZODIAC; a modified release plus patch file is required for reproducibility) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
sirius console --input dendroides.mzML --output zodiac_out --formula --zodiac  # 10 chains, 1,000 burn-in / 2,000 sampling epochs
```

## Evaluation signals

- Marginal file sanity: zodiac_dendroides_marginals.csv is a compounds × candidates matrix with every value in [0,1].
- Trajectory file sanity: zodiac_dendroides_trajectory.csv records epoch, total score, and identification rate per epoch; total score should stabilize after the 1,000-epoch burn-in.
- Ten parallel chains agree on high-scoring formulas; posterior marginals aggregated across thinned samples (every 10th state) are consistent across chains.
- Identification rate against ground truth (established by spectral library search and manual validation) improves over SIRIUS-alone, with a 16.2-fold reduction in incorrect annotations expected on dendroides-like data.
- Runtime check: with the fast O(|V(c)| + deg(u) + deg(v)) conditional update, an estimated ~25-fold speedup over a naive sampler should keep sampling swift on 40 cores.

## Limitations

- The optimized Gibbs sampler was not benchmarked against a naive version; the ~25-fold speedup is a theoretical estimate, not an empirical measurement.
- Anchors (library hits) contribute only a small effect to molecular formula annotations.
- SIRIUS 4 performance drops substantially for compounds above 500 Da; ZODIAC's gains are most distinctive on such challenging datasets (dendroides, compounds >700 Da).
- Reproducibility requires the modified OpenMS 2.4.0 build plus a separate patch file, not stock OpenMS.
- High-quality adduct annotations must be established during preprocessing, since they cannot be reliably deduced from isotope patterns and MS/MS data.

## Evidence

- [abstract] ZODIAC reranks SIRIUS' molecular formula candidates, combining fragmentation tree computation with Bayesian statistics using Gibbs sampling.: [evidence span withheld — anchor status: not_found]
- [results] ZODIAC full name and purpose: [evidence span withheld — anchor status: not_found]
- [results] estimated speedup not empirically evaluated: [evidence span withheld — anchor status: not_found]
