---
name: molecular-formula-candidate-reranking
description: Use when you have SIRIUS top-50 molecular formula candidate lists for
  an untargeted LC-MS/MS dataset and per-compound top-1 accuracy is insufficient (large
  compounds >500 Da, complex extracts), and you want dataset-aware Bayesian reranking.
when_to_use_negative:
- Input is a raw feature table or MS1-only data with no SIRIUS fragmentation trees
  and MS/MS spectra — ZODIAC's similarity edges require fragmentation trees of the
  monoisotopic-peak MS/MS spectra
- The dataset is a pure standard mix where SIRIUS alone already performs excellently
  (e.g. NIST1950), so reranking gains are marginal
- Compounds are small (<500 Da) where SIRIUS 4 alone shows excellent performance and
  ZODIAC's network signal adds little
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Computes fragmentation trees, isotope pattern analysis, and generates the
    top-50 molecular formula candidates that ZODIAC reranks; ZODIAC is implemented
    inside the SIRIUS software and run through its command-line tool
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Performs the Bayesian Gibbs-sampling reranking of SIRIUS candidates via the
    weighted assignment graph
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS (modified 2.4.0)
  role: Preprocesses input mzML/mzXML files and discards low-quality MS/MS spectra
    in the ZODIAC workflow
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:996d346a9cdac8ff9958b2e1f4f9c9d6ba558d2d4b960bb24bff89c7d4536409
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
  - OpenMS (modified 2.4.0)
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# molecular-formula-candidate-reranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reranks SIRIUS's top-50 molecular formula candidates per compound with ZODIAC's Bayesian, Gibbs-sampling-based model that couples fragmentation-tree similarity across the LC-MS/MS dataset. This drastically reduces incorrect formula annotations, e.g. a 16.2-fold error reduction on the dendroides plant-extract dataset.

## When to use

Use when you have SIRIUS top-50 molecular formula candidate lists for an untargeted LC-MS/MS dataset and per-compound top-1 accuracy is insufficient (large compounds >500 Da, complex extracts), and you want dataset-aware Bayesian reranking.

## When NOT to use

- Input is a raw feature table or MS1-only data with no SIRIUS fragmentation trees and MS/MS spectra — ZODIAC's similarity edges require fragmentation trees of the monoisotopic-peak MS/MS spectra
- The dataset is a pure standard mix where SIRIUS alone already performs excellently (e.g. NIST1950), so reranking gains are marginal
- Compounds are small (<500 Da) where SIRIUS 4 alone shows excellent performance and ZODIAC's network signal adds little

## Inputs

- SIRIUS top-50 molecular formula candidate lists per compound (with candidate scores) for an LC-MS/MS dataset
- SIRIUS fragmentation trees for each compound's MS/MS spectrum
- Peak list / mzML or mzXML files preprocessed with OpenMS (low-quality MS/MS spectra discarded)
- Optional: spectral library search hits used as anchors
- Optional: ground-truth adduct annotations from preprocessing (e.g. groundtruth_dendroides.csv) to constrain adducts

## Outputs

- Per-compound posterior marginals over formula candidates (compounds x candidates, values in [0,1]), e.g. zodiac_dendroides_marginals.csv
- Sampling trajectory table (epoch, total score, identification rate), e.g. zodiac_dendroides_trajectory.csv
- Top-1 reranked molecular formula annotation per compound
- Optionally, the weighted assignment graph (JSON/TSV) with estimated (mu, sigma^2, tau) parameters

## How to apply

First run SIRIUS to compute fragmentation trees and retain the top 50 molecular formula candidates per compound. Build ZODIAC's weighted assignment graph: one candidate node per formula per compound (plus a dummy node) with softmax likelihoods over the SIRIUS candidate scores, edges weighted by pairwise fragmentation-tree similarity s(u,v) normalized by weighted tree sizes with Pareto-based fragment weights. Estimate the log-normal p-value prior from 100,000 sampled non-zero similarity scores (mu = median, sigma^2 via MAD) and set the similarity threshold tau at the 95% quantile; add an anchor bonus psi for spectral library hits (min_l = 0.5, lambda = 1000, bonus reduced by 0.1 for analogue-mode hits). Run ZODIAC's Gibbs sampler as 10 parallel Markov chains with a burn-in of 1,000 epochs and 2,000 sampled epochs, resampling each compound's formula from its O(|V(c)| + deg(u) + deg(v)) conditional (Lemma 2), then aggregate thinned samples into per-compound posterior marginals and take the top-ranked formula.

## Related tools

- **SIRIUS** (Computes fragmentation trees, isotope pattern analysis, and generates the top-50 molecular formula candidates that ZODIAC reranks; ZODIAC is implemented inside the SIRIUS software and run through its command-line tool) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Performs the Bayesian Gibbs-sampling reranking of SIRIUS candidates via the weighted assignment graph) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS (modified 2.4.0)** (Preprocesses input mzML/mzXML files and discards low-quality MS/MS spectra in the ZODIAC workflow) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
sirius analyze --max-candidates 50 --zodiac --output zodiac_out dendroides.mgf
```

## Evaluation signals

- Posterior marginals table is a compounds x candidates matrix with every value in [0,1]
- Trajectory file records total score and identification rate per epoch over 1,000 burn-in + 2,000 sampled epochs across 10 parallel chains
- On dendroides with known adducts, expect SIRIUS top-1 accuracy ~67.17% (135/135-compound subset basis: 135 correct) versus ZODIAC ~99.50% (200 correct) — a large jump in identification rate
- Estimated prior parameters: mu equals the median of 100,000 sampled non-zero similarity scores, sigma^2 equals their MAD-based estimate, and tau equals their 95% quantile
- Graph validation: each compound has a dummy node and softmax node weights are normalized per compound

## Limitations

- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data; high-quality adduct annotations must be established during preprocessing or accuracy gains shrink
- Library-hit anchors have only a small effect on molecular formula annotations
- The optimized Gibbs sampler's ~25-fold speedup was estimated theoretically, not evaluated against a naive implementation
- Reproducibility requires a modified version of OpenMS 2.4.0 plus a separate patch file, not stock OpenMS

## Evidence
