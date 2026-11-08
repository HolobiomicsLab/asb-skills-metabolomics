---
name: mces-graph-distance-evaluation
description: Use when scoring de novo structure-elucidation outputs (SELFIES/SMILES
  candidates from spectra) against reference molecules and you need a structural-graph
  distance in addition to exact-match and fingerprint-similarity metrics, e.g. filling
  MCES columns of benchmark comparison tables.
when_to_use_negative:
- You only need exact structural identity — full InChIKey comparison is sufficient
  and MCES adds graph-distance computation overhead.
- You need a similarity score where higher is better and rewards bit overlap — maximum
  Tanimoto similarity (Morgan radius 2, 2048 bits) is the complementary metric.
- The candidate list has not been reduced/ranked (e.g. by formula distance), so 'minimum
  MCES over Top-k' is ill-defined.
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: RDKit
  role: cheminformatics toolkit used to convert predicted/reference SELFIES to Mol
    objects and compute structure-level metrics including MCES and Morgan fingerprints
- name: MS-Bart
  role: generates the candidate SELFIES molecules per spectrum whose Top-k minimum
    MCES is reported
  repo: github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_content_hash: sha256:3b019b2bb7c4ce62330edd50ecc4cc1af7d2b44698801e6362e2344bdfe0de53
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MS-BART
  - MS-Bart
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mces-graph-distance-evaluation

## Summary

Evaluate predicted molecular structures against ground-truth structures using the minimum common edge substructure (MCES) graph distance, reporting the minimum MCES over Top-1 and Top-10 candidate sets alongside InChIKey accuracy and Tanimoto similarity. This is one of the six key metrics MS-Bart reports on MassSpecGym and NPLIB1.

## When to use

Use when scoring de novo structure-elucidation outputs (SELFIES/SMILES candidates from spectra) against reference molecules and you need a structural-graph distance in addition to exact-match and fingerprint-similarity metrics, e.g. filling MCES columns of benchmark comparison tables.

## When NOT to use

- You only need exact structural identity — full InChIKey comparison is sufficient and MCES adds graph-distance computation overhead.
- You need a similarity score where higher is better and rewards bit overlap — maximum Tanimoto similarity (Morgan radius 2, 2048 bits) is the complementary metric.
- The candidate list has not been reduced/ranked (e.g. by formula distance), so 'minimum MCES over Top-k' is ill-defined.

## Inputs

- Top-k candidate SELFIES or SMILES structures per spectrum (k=10 reporting ranks from 100 sampled candidates)
- Ground-truth molecular structures of the benchmark test fold (MassSpecGym, NPLIB1)
- Formula-distance-ranked prediction lists (ranked by atom-count differences, ties by log-probability)

## Outputs

- Per-benchmark Top-1 and Top-10 minimum MCES distances (lower is better)
- Rows assembled into comparison tables such as table8_temperature_sensitivity.csv and epsilon_sensitivity_results.csv

## How to apply

Convert the generated candidate SELFIES strings and the reference molecules into RDKit Mol objects, then compute the MCES graph distance between each prediction and its reference structure using RDKit. For spectra with k generated candidates (100 sampled candidates per spectrum via beam-search multinomial sampling), select the minimum MCES across the candidate set for both Top-1 and Top-10 ranks, after candidates have been re-ranked by formula distance (sum of absolute atom-count differences) with ties broken by estimated log-probability. Report MCES as a distance where lower is better (downward arrows in benchmark tables), aggregated across the full test fold — e.g. MassSpecGym with all adducts retained, or after the [M+Na]+ filtering that retains only [M+H]+ for fine-tuning and alignment. Emit the per-setting rows into comparison CSVs (e.g. table8_temperature_sensitivity.csv, epsilon_sensitivity_results.csv) together with Top-1/Top-10 InChIKey accuracy and maximum Tanimoto similarity from Morgan fingerprints (radius 2, 2048 bits). MCES also doubles as a data-filtering statistic: pretraining pairs are excluded when their MCES distance is less than two from any test-fold molecule.

## Related tools

- **RDKit** (cheminformatics toolkit used to convert predicted/reference SELFIES to Mol objects and compute structure-level metrics including MCES and Morgan fingerprints)
- **MS-Bart** (generates the candidate SELFIES molecules per spectrum whose Top-k minimum MCES is reported) — github.com/OpenDFM/MS-BART

## Examples

```
bash scripts/msg/eval.sh  # evaluates Top-1/Top-10 exact-match (full InChIKey), minimum MCES, and maximum Tanimoto on the MassSpecGym test fold
```

## Evaluation signals

- Directionality check: MCES is reported with a downward arrow (lower is better); a correct run yields Top-10 minimum MCES ≤ Top-1 minimum MCES (e.g. NPLIB1 table shows Top-1 MCES values larger than Top-10 for every model).
- Reference anchors: epsilon=0.11 on MassSpecGym gives Top-1 MCES ≈ 16.47, and metrics remain essentially constant across sampling temperatures 0.2/0.4/0.8 (Table 8 temperature robustness).
- Cross-model sanity: MCES must decrease monotonically in the benchmark tables where accuracy increases (e.g. stronger models show lower Top-1 MCES than Spec2Mol's 48.57).
- Consistency across the metric triplet: Top-k accuracy (full InChIKey), minimum MCES, and maximum Tanimoto should be computed on the same ranked candidate lists.

## Limitations

- Fingerprint predictions from the external MIST model cap downstream quality, which propagates into MCES values.
- Benchmark comparisons are constrained by dataset licensing/filtering: NIST23 access is restricted and MassSpecGym/NPLIB1 splits with [M+Na]+ filtered out change reported numbers.
- Revised MIST encoders (MIST 4M-aug) supersede earlier checkpoints; MCES values from prior releases are not directly comparable.

## Evidence

- [other] computing Top-1/Top-10 exact-match accuracy (full InChIKey), minimum MCES, and maximum Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits, RDKit): [evidence span withheld — anchor status: not_found]
- [results] the reported Top-1/Top-10 accuracy, MCES and Tanimoto similarity metrics in Table 8 are essentially constant: [evidence span withheld — anchor status: not_found]
- [readme] | Model | Top-1 Acc ↑ | Top-1 MCES ↓ | Top-1 Tanimoto ↑ | Top-10 Acc ↑ | Top-10 MCES ↓ | Top-10 Tanimoto ↑ |: [evidence span withheld — anchor status: external_doc]
