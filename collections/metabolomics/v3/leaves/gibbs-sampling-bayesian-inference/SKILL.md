---
name: gibbs-sampling-bayesian-inference
description: Use when SIRIUS top-ranked molecular formulas are untrusted — e.g. metabolites
  above 500 Da in a compound network from untargeted LC-MS/MS — and top-50 candidate
  lists plus adduct assignments are available for network-based reranking.
when_to_use_negative:
- Datasets where SIRIUS already shows excellent performance and little headroom exists,
  as on the NIST1950 and tomato datasets
- A single isolated compound with no network of related compounds — ZODIAC's Bayesian
  inference relies on the compound similarity network
- Cases where high-quality adduct annotations have not been established during preprocessing,
  since ZODIAC depends on them
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SIRIUS
  role: Computes fragmentation trees and supplies the top 50 molecular formula candidates;
    ZODIAC is implemented inside the SIRIUS CLI
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: ZODIAC
  role: Performs the Gibbs sampling-based Bayesian reranking of SIRIUS candidates
    over the compound similarity network
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS 2.4.0 (modified)
  role: Preprocessing of mzML/mzXML input and discarding low-quality MS/MS spectra;
    a modified version is required for the ZODIAC workflow
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:ac6becaee28b2a3f0bfe4739abfe596393c31d0ebd7daf46d5d5e75a41051956
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - OpenMS 2.4.0 (modified)
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gibbs-sampling-bayesian-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

ZODIAC-style Gibbs sampling Bayesian reranking of SIRIUS molecular formula candidates: it takes the top 50 SIRIUS candidates per compound plus a compound-similarity network and, optionally, spectral-library anchors, to correct formula annotations in untargeted LC-MS/MS metabolomics.

## When to use

Use when SIRIUS top-ranked molecular formulas are untrusted — e.g. metabolites above 500 Da in a compound network from untargeted LC-MS/MS — and top-50 candidate lists plus adduct assignments are available for network-based reranking.

## When NOT to use

- Datasets where SIRIUS already shows excellent performance and little headroom exists, as on the NIST1950 and tomato datasets
- A single isolated compound with no network of related compounds — ZODIAC's Bayesian inference relies on the compound similarity network
- Cases where high-quality adduct annotations have not been established during preprocessing, since ZODIAC depends on them

## Inputs

- SIRIUS molecular formula candidate lists (top 50 candidates per compound with fragmentation tree scores)
- Per-compound ionization adduct assignments from preprocessing (e.g. groundtruth_dendroides.csv)
- LC-MS/MS peak lists / MS/MS spectra of metabolites
- Optional: spectral library search hits usable as anchors

## Outputs

- ZODIAC top-ranked molecular formula per compound (reranked SIRIUS candidates)
- Compound similarity network over molecular formulas
- Accuracy comparison table, e.g. dendroides_known_adducts_accuracy.csv

## How to apply

First run SIRIUS to compute fragmentation trees and produce the top 50 molecular formula candidates per compound (ZODIAC builds on these as its candidate pool). Constrain each compound's ionization adduct from high-quality preprocessing (e.g. ground-truth adducts from groundtruth_dendroides.csv), since adducts cannot be reliably deduced from isotope pattern and MS/MS data. Run ZODIAC's Gibbs sampling via the SIRIUS command-line tool: it combines fragmentation tree scores with Bayesian statistics over the compound similarity network, using ten parallel chains, a burn-in of 1,000 epochs and 2,000 sampling epochs (the paper's timing setup on 40 cores). Optionally integrate spectral library search hits as anchors in the stochastic model, though anchors contribute only a small effect. Extract the top-ranked formula per compound and compare against a ground truth established by spectral library search and manual validation.

## Related tools

- **SIRIUS** (Computes fragmentation trees and supplies the top 50 molecular formula candidates; ZODIAC is implemented inside the SIRIUS CLI) — https://bio.informatik.uni-jena.de/software/sirius/
- **ZODIAC** (Performs the Gibbs sampling-based Bayesian reranking of SIRIUS candidates over the compound similarity network) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS 2.4.0 (modified)** (Preprocessing of mzML/mzXML input and discarding low-quality MS/MS spectra; a modified version is required for the ZODIAC workflow) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- Every ZODIAC top-ranked formula must be one of the original top 50 SIRIUS candidates for that compound
- On the dendroides dataset with known adducts, expect SIRIUS ~67.17% (135/200 compounds correct) and ZODIAC ~99.50% (200/200), i.e. a 66-fold error rate decrease
- Under the paper's default conditions, ZODIAC should decrease incorrect annotations ~16.2-fold versus SIRIUS on the challenging dendroides plant extract
- ZODIAC accuracy should never fall below SIRIUS accuracy on the same candidate lists and adduct constraints
- Chains should be run with the specified sampling configuration (10 parallel chains, 1,000 burn-in epochs, 2,000 sampling epochs); outputs should be stable across chains

## Limitations

- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data; it must be established during preprocessing
- Library-hit anchors have only a small effect on molecular formula annotations
- SIRIUS (and hence the candidate pool) has problems annotating molecular formulas for compounds above 500 Da
- The optimized Gibbs sampler's ~25-fold speedup was only estimated theoretically, not evaluated against a naive version
- Reproducibility requires a modified OpenMS 2.4.0 plus a patch file, not stock OpenMS

## Evidence

- [discussion] High-quality adduct annotations must be established during preprocessing.: [evidence span withheld — anchor status: not_found]
