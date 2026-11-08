---
name: error-rate-reduction-analysis
description: Use when benchmarking molecular formula annotation accuracy of SIRIUS
  versus ZODIAC against a manually validated ground truth across LC-MS/MS datasets
  such as dendroides, NIST1950, tomato, diatoms, and mice stool.
when_to_use_negative:
- When no manually validated ground truth exists for the evaluated compounds — accuracy
  cannot be computed against spectral library search plus manual validation
- When the goal is de novo structure or fingerprint identification (CSI:FingerID)
  rather than molecular formula annotation accuracy benchmarking
- When evaluating a single compound or spectrum rather than computing dataset-level
  accuracy and error-rate ratios
tools:
- name: SIRIUS
  role: Computes top-ranked molecular formula annotations and fragmentation trees
    that serve as the baseline accuracy in the benchmark
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Reranks the top 50 SIRIUS formula candidates using Bayesian statistics via
    Gibbs sampling; its accuracy is compared against SIRIUS to derive the error-rate
    reduction
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS 2.4.0 (modified)
  role: Preprocessing of input mzML/mzXML files with discarding of low-quality MS/MS
    spectra before annotation
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:22a1c41794ac13bac02fae863115a9d8a6f497bd36ad44d0e93cc81a905a623d
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# error-rate-reduction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks SIRIUS top-ranked molecular formula annotations against ZODIAC's Gibbs-sampling reranking of the top 50 SIRIUS candidates, computing per-dataset accuracy and incorrect-annotation rates to quantify error-rate reduction (e.g. the 16.17-fold improvement on dendroides).

## When to use

Use when benchmarking molecular formula annotation accuracy of SIRIUS versus ZODIAC against a manually validated ground truth across LC-MS/MS datasets such as dendroides, NIST1950, tomato, diatoms, and mice stool.

## When NOT to use

- When no manually validated ground truth exists for the evaluated compounds — accuracy cannot be computed against spectral library search plus manual validation
- When the goal is de novo structure or fingerprint identification (CSI:FingerID) rather than molecular formula annotation accuracy benchmarking
- When evaluating a single compound or spectrum rather than computing dataset-level accuracy and error-rate ratios

## Inputs

- Deposited SIRIUS molecular formula annotation results (top-ranked formulas per compound)
- Deposited ZODIAC reranking results (reranked top 50 SIRIUS candidates per compound)
- Ground-truth formula files from spectral library search and manual validation for five datasets (dendroides MSV000080502, NIST1950 MSV000081364, tomato, diatoms, mice stool)
- Preprocessed mzML/mzXML files processed with modified OpenMS 2.4.0 (low-quality MS/MS spectra discarded)

## Outputs

- CSV benchmark table of 5 datasets x 2 methods with per-dataset annotation accuracy and incorrect-annotation rate columns
- Per-dataset error-reduction factor (ratio of SIRIUS to ZODIAC incorrect-annotation rates, e.g. 16.17-fold on dendroides)

## How to apply

Download the deposited SIRIUS and ZODIAC annotation results and ground-truth formula files from https://bio.informatik.uni-jena.de/data/. Parse the annotation files to extract, per compound and per dataset, the top-ranked formula from SIRIUS (which enumerates formula candidates) and from ZODIAC (which reranks the top 50 SIRIUS candidates via Gibbs sampling over the similarity network). Match each top-ranked formula against the ground truth established by spectral library search (NIST17 library) plus manual validation for the five MassIVE datasets (dendroides MSV000080502, NIST1950 MSV000081364, tomato, diatoms, mice stool). Compute per-dataset accuracy as the fraction of correct top-ranked formulas and the incorrect-annotation rate for each method, then derive the error-reduction factor as the ratio of SIRIUS to ZODIAC incorrect-annotation rates. Expect the largest improvement on the challenging dendroides plant extract where most compounds exceed 700 Da and SIRIUS performance degrades above 500 Da.

## Related tools

- **SIRIUS** (Computes top-ranked molecular formula annotations and fragmentation trees that serve as the baseline accuracy in the benchmark) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Reranks the top 50 SIRIUS formula candidates using Bayesian statistics via Gibbs sampling; its accuracy is compared against SIRIUS to derive the error-rate reduction) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS 2.4.0 (modified)** (Preprocessing of input mzML/mzXML files with discarding of low-quality MS/MS spectra before annotation) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- Dendroides accuracy must be 50.76% correct for SIRIUS vs 96.95% for ZODIAC on 197 evaluable ground truth compounds
- Dendroides error-reduction factor must equal the ratio of incorrect-annotation rates, reproducing the reported ~16.17-fold (16.2-fold) decrease
- Other datasets must reproduce: NIST1950 8.51% vs 5.32% incorrect, tomato 4.44% vs 2.22%, diatoms ~12.90% roughly halved, mice stool 30.23% vs 20.93%
- Output table must contain 5 datasets x 2 methods with both accuracy and error-rate columns
- Matched compound counts must correspond to evaluable ground-truth compounds (e.g. 197 for dendroides)

## Limitations

- No distinct difference was found between the cosine score of ZODIAC's correct and incorrect molecular formula assignments, so cosine score cannot be used to filter out errors
- A complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible
- Reproducibility requires a modified version of OpenMS 2.4.0 plus a separate patch file, not stock OpenMS
- Library-hit anchors contribute only a small effect to molecular formula annotations, so anchor-based gains should not be overstated

## Evidence

- [readme] SIRIUS is the software framework for LC-MS/MS small-molecule analysis integrating ZODIAC: [evidence span withheld — anchor status: external_doc]
