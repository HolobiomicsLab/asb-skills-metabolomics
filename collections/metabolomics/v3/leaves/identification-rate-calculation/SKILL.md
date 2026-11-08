---
name: identification-rate-calculation
description: Use when benchmarking a formula/structure identification tool on the
  CASMI 2016 LC-MS/MS challenge data and you need to report fraction of correct answers
  at rank 1 and rank k, mirroring the SIRIUS 4 reanalysis protocol.
when_to_use_negative:
- The goal is running SIRIUS/CSI:FingerID itself (formula ranking or structure search)
  rather than scoring a completed set of predictions against CASMI reference answers.
- The evaluation must include compounds that overlap with CSI:FingerID's training
  data — the structure-disjointness filter required by the published protocol would
  remove them.
- The input is already an aggregated identification-rate table; only per-challenge
  rank data can be recomputed into top-k rates.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Ranks candidate molecular formulas from isotope patterns and MS/MS fragmentation
    trees, producing the top-1/top-2 formula ranks that are scored
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Produces ranked candidate structures searched in molecular structure databases,
    whose top-10 (and top-3 biomolecular-database) ranks are scored
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:da3c92048befb85ae4742b2089005e4e2e0a2ccdd39d80651fc7695da2a8bd75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - SIRIUS
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 36
---

# identification-rate-calculation

## Summary

Computes top-k identification rates (top 1/top 2 for molecular formulas, top 10 for structures, top 3 per database) from SIRIUS 4 / CSI:FingerID reanalysis output, after excluding challenges whose correct structure is not structure-disjoint from CSI:FingerID's training data.

## When to use

Use when benchmarking a formula/structure identification tool on the CASMI 2016 LC-MS/MS challenge data and you need to report fraction of correct answers at rank 1 and rank k, mirroring the SIRIUS 4 reanalysis protocol.

## When NOT to use

- The goal is running SIRIUS/CSI:FingerID itself (formula ranking or structure search) rather than scoring a completed set of predictions against CASMI reference answers.
- The evaluation must include compounds that overlap with CSI:FingerID's training data — the structure-disjointness filter required by the published protocol would remove them.
- The input is already an aggregated identification-rate table; only per-challenge rank data can be recomputed into top-k rates.

## Inputs

- CC-BY CASMI 2016 LC-MS/MS challenge spectra (positive and negative ionization modes)
- Reference molecular formulas and structures per CASMI challenge
- SIRIUS-ranked candidate molecular formula lists per challenge
- CSI:FingerID-ranked candidate structure lists per challenge (all databases and biomolecular database)
- CSI:FingerID training structure sets (predictor=1 positive mode, predictor=2 negative mode)

## Outputs

- Per-challenge rank of the correct molecular formula (top 1 / top 2)
- Per-challenge rank of the correct structure (top 10; top 3 for the biomolecular database)
- casmi2016_results_table.csv with per-mode and per-database identification rates

## How to apply

Download the CC-BY CASMI 2016 re-evaluation LC-MS/MS challenge data from bio.informatik.uni-jena.de/data and extract per-challenge spectra with their reference formulas and structures. Run SIRIUS isotope pattern analysis and fragmentation tree computation per spectrum to rank candidate molecular formulas in positive and negative ionization modes, then upload fragmentation trees and spectra to CSI:FingerID to retrieve ranked candidate structures. Restrict a repeat CSI:FingerID search to the biomolecular structure database for positive-mode challenges. Apply the key filter: exclude challenges whose correct structure is not structure-disjoint from CSI:FingerID's training data (training structures are available via the CSI:FingerID web API for predictor=1, positive mode, and predictor=2, negative mode). Finally, compute top-1 and top-k rates (top 2 for formulas, top 10 for structures, top 3 for the biomolecular database), assemble per-mode and per-database rates into a results table, and validate each recomputed rate against the published reanalysis (93.8% correct formulas at top 1, 96.2% at top 2; >70% identification rates overall) within rounding tolerance.

## Related tools

- **SIRIUS 4** (Ranks candidate molecular formulas from isotope patterns and MS/MS fragmentation trees, producing the top-1/top-2 formula ranks that are scored) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Produces ranked candidate structures searched in molecular structure databases, whose top-10 (and top-3 biomolecular-database) ranks are scored) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Recomputed correct-formula top-1 rate equals 93.8% and top-2 rate equals 96.2% on the CASMI 2016 reanalysis within rounding tolerance.
- Overall recomputed identification rates exceed 70%, consistent with the published claim on challenging metabolomics datasets.
- Every excluded challenge is verifiably not structure-disjoint from the CSI:FingerID training structures fetched via the trainingstructures API (predictor=1 or 2).
- casmi2016_results_table.csv contains separate rate rows per ionization mode and per searched database (all databases vs. biomolecular database).
- Per-challenge rank values fall within the expected k cutoffs (formula rank ≤ 2; structure rank ≤ 10; biomolecular-database rank ≤ 3).

## Limitations

- Results are only comparable to the published values after applying the structure-disjointness filter against CSI:FingerID's training data.
- CSI:FingerID's web services are for academic research and education use only; commercial use requires a Bright Giant GmbH license.
- The SIRIUS isotope pattern analysis is described as 'hopefully more stable than before' after being rewritten, and in-source fragmentations/adduct support is labeled experimental — both can affect reproducibility of ranks.

## Evidence

- [results] In the CASMI 2016 reanalysis, SIRIUS 4 assigned the correct molecular formula to 93.8% of the compounds using MS/MS and isotope pattern data, and 96.2% in the top 2.: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract identification-rate claim: "Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets."
- [methods] Isotope pattern analysis method: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [readme] Training structures API endpoint in README: "https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1 (training structures for positive ion mode)"
