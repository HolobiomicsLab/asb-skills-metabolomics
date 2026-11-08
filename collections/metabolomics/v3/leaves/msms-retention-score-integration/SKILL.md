---
name: msms-retention-score-integration
description: Use when candidate lists from MS/MS tools (SIRIUS, MS-DIAL) for LC-MS
  peaks in an RPLC run need re-ranking with predicted retention-order scores, or when
  peaks lack usable MS/MS spectra.
when_to_use_negative:
- The chromatography is HILIC or another non-RPLC system — ROASMI applies only to
  RPLC and retraining HILIC models with generalizability remains future work
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S — the
  trained models do not cover the broader chemical space
- Candidate pairs are indistinguishable by the retention model (e.g. 4-hydroxybenzoic
  acid vs 2,5-dihydroxybenzoic acid, close retention with high prediction variance)
  — such pairs should be excluded rather than resolved by retention ordering
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3336
tools:
- name: ROASMI
  role: Predicts retention-order scores and uncertainty (variance) for each candidate
    structure in the ready-to-identify lists via D-MPNN molecular embedding plus RankNet
  repo: https://github.com/FangYuan717/ROASMI
- name: msms_rt_score_integration
  role: Provides the probabilistic framework used by Identify.py to combine predicted
    retention scores with MS/MS scores for candidate ranking
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: chemprop
  role: Backbone of ROASMI's molecular embedding module (D-MPNN) that maps molecules
    directly from structure
  repo: https://github.com/chemprop/chemprop
- name: pandas
  role: Handles CSV candidate lists and per-peak re-ranked output tables (reranked_topk_results.csv)
- name: PyTorch
  role: Deep-learning backend for ROASMI model training and inference (GPU-enabled
    install supported)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:d2c4e0ee76bc662e29b9aa0e1c454c3672399dc9ac835c47f3dc700a1f199b32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - msms_rt_score_integration
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# msms-retention-score-integration

## Summary

A probabilistic integration of ROASMI-predicted retention-order scores with MS/MS scores (SIRIUS, MS-DIAL) to re-rank candidate structures, improving candidate ranking and top-k identification accuracy over MS/MS tools alone.

## When to use

Use when candidate lists from MS/MS tools (SIRIUS, MS-DIAL) for LC-MS peaks in an RPLC run need re-ranking with predicted retention-order scores, or when peaks lack usable MS/MS spectra.

## When NOT to use

- The chromatography is HILIC or another non-RPLC system — ROASMI applies only to RPLC and retraining HILIC models with generalizability remains future work
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S — the trained models do not cover the broader chemical space
- Candidate pairs are indistinguishable by the retention model (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid, close retention with high prediction variance) — such pairs should be excluded rather than resolved by retention ordering

## Inputs

- Ready-to-identify candidate lists from MS/MS tools (SIRIS and MS-DIAL outputs) as CSV files in the 'data' folder (e.g. identify_toy_set.csv)
- Pre-trained ROASMI models (ROASMI_1–ROASMI_5 ensemble) for RPLC at eluent pH ~2.7
- MS/MS scores per candidate (e.g. SIRIUS or MS-DIAL scores)

## Outputs

- Per-peak re-ranked candidate tables with integrated MS/MS + retention scores
- Top-k accuracy metrics (top-1, top-3, top-5) for MS/MS-only vs ROASMI-integrated rankings
- Candidate-reduction statistics (e.g. 87.3% of candidates removed by retention ordering)

## How to apply

Load the ready-to-identify candidate lists (MS-DIAL and SIRIUS outputs) as CSV files in the 'data' folder — e.g. the 106 CASMI 2022 priority challenges and the 49 correctly-inferred-formula peaks of dataset_38. Run ROASMI_predict.py (PredictArgs in args.py) to compute ROASMI retention scores for every candidate; the ensemble (ROASMI_1–ROASMI_5) yields prediction variance used to flag indistinguishable pairs, which should be excluded from downstream identification. Then run Identify.py (IdentifyArgs in args.py) to combine predicted retention scores with MS/MS scores using the probabilistic framework of msms_rt_score_integration (Bach et al., 2020). Evaluate by computing top-k accuracy (top-1, top-3, top-5) for the MS/MS-only baseline versus the ROASMI-integrated ranking, and quantify candidate reduction from retention ordering. The rationale: retention order is far more replicable across runs than absolute RT, so ranking candidates by predicted elution order prunes implausible candidates (~87.3% removed) and raises the rank of the correct structure.

## Related tools

- **ROASMI** (Predicts retention-order scores and uncertainty (variance) for each candidate structure in the ready-to-identify lists via D-MPNN molecular embedding plus RankNet) — https://github.com/FangYuan717/ROASMI
- **msms_rt_score_integration** (Provides the probabilistic framework used by Identify.py to combine predicted retention scores with MS/MS scores for candidate ranking) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **chemprop** (Backbone of ROASMI's molecular embedding module (D-MPNN) that maps molecules directly from structure) — https://github.com/chemprop/chemprop
- **pandas** (Handles CSV candidate lists and per-peak re-ranked output tables (reranked_topk_results.csv))
- **PyTorch** (Deep-learning backend for ROASMI model training and inference (GPU-enabled install supported))

## Examples

```
python code/ROASMI_predict.py && python code/Identify.py
```

## Evaluation signals

- Re-ranked table must reproduce the reported 87.3% candidate-reduction figure when retention ordering is applied (Supplementary Table 3)
- Top-k accuracy pattern must match Fig. 6c-f: ROASMI integration improves average rank of the correct structure by ~2 units vs SIRIUS alone and ~12 units vs MS-DIAL alone
- Ensemble variance across ROASMI_1–ROASMI_5 should be computed; pairs with high variance (~9% for 81% elution-order probability) flagged as distinguishable only above the reliability threshold
- Integrated rankings must be checked against the MS/MS-only baseline for monotone top-k improvement rather than degradation on the toy sets (identify_toy_set.csv)

## Limitations

- ROASMI applies only to RPLC systems at eluent pH ~2.7; no HILIC support yet
- Limited to conventional compounds within elemental restrictions of C, H, N, O, P, and S
- Retention order of very close chromatographic peaks may be inconsistent across experiments; the model cannot distinguish certain pairs (e.g. 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid)
- RT outliers from slight changes in experimental conditions remain unresolved and require mechanistic study

## Evidence

- [methods] Integrate ROASMI with MS/MS scores: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [abstract] Compatibility with other scorers: "ROASMI is flexible enough to be retrained with user-defined reference sets and is compatible with other MS/MS scorers"
- [readme] Probabilistic integration framework: "Probabilistic Framework for Integration of Mass Spectrum and Retention Time Information in Small Molecule Identification"
- [readme] RankNet retention module: "This module is an extension of chemprop described in the paper [Analyzing Learned Molecular Representations for Property Prediction]"
- [discussion] Excluding indistinguishable pairs: "Such pairs should be excluded from the subsequent small molecule identification process."
