---
name: tandem-ms-retrieval-metric-computation
description: 'Reference for evaluating a de novo structure generator on MS2 spectra:
  you need the retrieval rate and top-1 rate against a holdout set, plus the known
  ablation numbers (31%/17% naive vs. 45%/25% fingerprint-guided on GNPS) as a baseline
  to beat.'
when_to_use_negative:
- Input is a biological sample's feature list rather than a benchmark with known structures
  — the metric requires ground-truth annotations.
- You want similarity-based quality when no correct structure exists in the candidate
  pool (e.g. novel natural products); use the best-in-training-set or best-database-candidate
  Tanimoto comparison instead, not retrieval rate.
- Spectra lack the ≥3 fragment ions needed for SIRIUS/CSI:FingerID processing or were
  not run through the formula filter — the candidate pool will be empty or unreliable,
  invalidating the rate.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSNovelist
  role: Source of the de novo candidate SMILES and modified-Platt re-ranked scores
    that feed the retrieval and top-1 metric computation
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: Predicts the molecular formula constraint under which candidates are generated,
    and its workspace is the input to MSNovelist
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CSI:FingerID
  role: Predicts the structural fingerprint xF used for the modified-Platt re-ranking
    that determines top-1 ranking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:6f81f69fc1222bd0c7f4cb8d213d2db6d90073e89831eafeefa2b09b0278d509
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tandem-ms-retrieval-metric-computation

## Summary

Compute and interpret structure-retrieval metrics (correct structure ranked first vs. retrieved anywhere among k candidates) for tandem-MS de novo pipelines, following MSNovelist's evaluation on GNPS and CASMI benchmarks.

## When to use

Reference for evaluating a de novo structure generator on MS2 spectra: you need the retrieval rate and top-1 rate against a holdout set, plus the known ablation numbers (31%/17% naive vs. 45%/25% fingerprint-guided on GNPS) as a baseline to beat.

## When NOT to use

- Input is a biological sample's feature list rather than a benchmark with known structures — the metric requires ground-truth annotations.
- You want similarity-based quality when no correct structure exists in the candidate pool (e.g. novel natural products); use the best-in-training-set or best-database-candidate Tanimoto comparison instead, not retrieval rate.
- Spectra lack the ≥3 fragment ions needed for SIRIUS/CSI:FingerID processing or were not run through the formula filter — the candidate pool will be empty or unreliable, invalidating the rate.

## Inputs

- Held-out MS2 spectra (MGF or SIRIUS-format .ms files) with known correct structures
- Per-query list of k dereplicated candidate SMILES with scores (e.g. decode-RUNID.csv with columns query, SMILES, score_mod_platt, rank_score_lim_mod_platt)
- Reference correct-structure annotations per query (SMILES or InChI) for the benchmark set
- Dataset-specific subsets (e.g. GNPS, GNPS-OK, CASMI, CASMI-OK) where spectra were filtered by molecular formula confidence

## Outputs

- Per-dataset retrieval rate (correct structure anywhere among candidates, %)
- Per-dataset top-1 rate (correct structure ranked first, %)
- Comparison table against fingerprint-guided and naive ablation baselines

## How to apply

For each held-out query MS2 spectrum, run the full pipeline (SIRIUS formula prediction, CSI:FingerID fingerprint prediction, structure generation under the molecular formula constraint, validation and dereplication of the k generated candidates, re-ranking by the modified Platt score against the query fingerprint xF), then check whether the correct structure (1) appears anywhere among the k dereplicated candidates ('retrieved') and (2) occupies rank 1 ('ranked first'). Aggregate these two rates per dataset and, where applicable, per subset (e.g. GNPS-OK, CASMI-OK, where spectra were filtered by molecular formula confidence). Report the two rates side by side, because the retrieval rate is the upper bound of what a database search could recover and the top-1 rate reflects the final re-ranking quality. Compare against both the fingerprint-guided MSNovelist numbers (GNPS 45% retrieved / 25% top-1; CASMI 57% retrieved / 26% top-1) and the naive fingerprint-free ablation (GNPS 31% / 17%; CASMI-OK 56% / 51%) to quantify the contribution of the fingerprint input. For the retrieval calculation, ensure the candidate pool is dereplicated so duplicate SMILES do not inflate the count; for top-1, sort strictly by descending score_mod_platt and take the first row per query.

## Related tools

- **MSNovelist** (Source of the de novo candidate SMILES and modified-Platt re-ranked scores that feed the retrieval and top-1 metric computation) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Predicts the molecular formula constraint under which candidates are generated, and its workspace is the input to MSNovelist) — https://bio.informatik.uni-jena.de/software/sirius/
- **CSI:FingerID** (Predicts the structural fingerprint xF used for the modified-Platt re-ranking that determines top-1 ranking)

## Examples

```
docker run -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf
```

## Evaluation signals

- For each query, the same number k of dereplicated candidates is present (no NaNs in the SMILES column) and rank_score_lim_mod_platt==1 marks exactly one row per query.
- Retrieval rate should fall between the naive ablation and fingerprint-guided values when comparing a modified pipeline (e.g. ≥31% on GNPS if fingerprint guidance is partially preserved).
- Top-1 rate must be ≤ retrieval rate for the same dataset; a violation indicates a re-ranking or rank-aggregation bug.
- Expected reference values to reproduce: GNPS 45% retrieved / 25% top-1; CASMI 57% retrieved / 26% top-1; naive GNPS 31% / 17%; naive CASMI-OK 56% / 51%.
- For 169 of 224 bryophyte spectra, MSNovelist's structure should score higher than the best database candidate (75% of cases) — a sanity check on the scoring scale.

## Limitations

- Correct structure may not be retrievable at all when isomers fragment indistinguishably — structural elucidation from MS2 is clearly not possible in all cases.
- MSNovelist is currently only trained for positive-mode data; negative-mode MS2 spectra are expected to yield lower retrieval and top-1 rates.
- Formula determination error rates rise to >50% for compounds with m/z up to 800 when considering individual spectra, capping achievable retrieval on high-mass queries.
- The model is trained on 1,232,184 known structures and may be unable to discover chemistry very different from known molecules.
- At least three fragment ions per MS2 spectrum are required; spectra with fewer will produce no usable candidates.

## Evidence

- [abstract] The correct structure was retrieved for 45% and ranked first for 25% of the instances: "The correct structure was retrieved for 45% and ranked first for 25% of the instances (Fig. 2a)."
- [abstract] naïve generation retrieved only 31% of all correct structures, and 17% were ranked first: "naïve generation retrieved only 31% of all correct structures, and 17% were ranked first"
- [abstract] MSNovelist correctly predicted 26% and retrieved 57% of structures: "Similarly, for the CASMI 2016 challenge, MSNovelist correctly predicted 26% and retrieved 57% of structures"
- [abstract] the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [readme] Order the results by score_mod_platt, descendingly, to get the top candidate: [evidence span withheld — anchor status: not_found]
- [results] we obtained a molecular formula with high confidence (≥80% explained peaks, ≥90% explained intensity, ≥0.9 ZODIAC score): "we obtained a molecular formula with high confidence (≥80% explained peaks, ≥90% explained intensity, ≥0.9% ZODIAC score)"
- [abstract] the k structures are validated and dereplicated: "the k structures are validated and dereplicated"
