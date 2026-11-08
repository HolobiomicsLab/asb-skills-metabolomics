---
name: retention-score-integrated-candidate-reranking
description: Use when ready-to-identify candidate lists from MS/MS tools (e.g. SIRIUS
  or MS-DIAL outputs as CSV) exist for RPLC-MS/MS peaks and retention-order scores
  should be integrated to improve top-k accuracy.
when_to_use_negative:
- The chromatography is HILIC — ROASMI currently applies only to RPLC systems and
  HILIC models with generalizability are future work
- Candidates fall outside the elemental restrictions of C, H, N, O, P, and S or otherwise
  leave ROASMI's application domain
- A peak's candidate pairs are retention-indistinguishable (chromatographic peaks
  close enough that order is inconsistent across experiments) — such pairs should
  be excluded rather than ranked
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3318
tools:
- name: ROASMI
  role: Predicts retention-order scores for every candidate in the ready-to-identify
    lists via ROASMI_predict.py; ensemble variance quantifies uncertainty
  repo: https://github.com/FangYuan717/ROASMI
- name: msms_rt_score_integration
  role: Provides the probabilistic framework (msms_rt_score_integration) used by Identify.py
    to combine predicted retention scores with MS/MS scores
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: chemprop
  role: Backbone of ROASMI's molecular embedding module (D-MPNN) that maps candidates
    directly into chemical spaces tuned to retention behavior
  repo: https://github.com/chemprop/chemprop
- name: SIRIUS
  role: MS/MS scorer whose candidate ranking is re-ranked (average rank of the correct
    structure improves by ~2 units)
- name: MS-DIAL
  role: MS/MS scorer whose candidate ranking is re-ranked (average rank of the correct
    structure improves by ~12 units)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:8b44f78a330e373ba0e9ea3f752b0670f087a7db3ebd1f5e12630063d7808fd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - MS-DIAL
  - PyTorch
  - ROASMI
  - SIRIUS
  - chemprop
  - msms_rt_score_integration
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# Retention-score-integrated candidate reranking

## Summary

Re-ranks candidate lists produced by MS/MS identification tools (SIRIUS, MS-DIAL) by combining ROASMI-predicted retention-order scores with MS/MS scores using the probabilistic integration framework of msms_rt_score_integration, improving top-k identification accuracy on RPLC metabolomics peaks.

## When to use

Use when ready-to-identify candidate lists from MS/MS tools (e.g. SIRIUS or MS-DIAL outputs as CSV) exist for RPLC-MS/MS peaks and retention-order scores should be integrated to improve top-k accuracy.

## When NOT to use

- The chromatography is HILIC — ROASMI currently applies only to RPLC systems and HILIC models with generalizability are future work
- Candidates fall outside the elemental restrictions of C, H, N, O, P, and S or otherwise leave ROASMI's application domain
- A peak's candidate pairs are retention-indistinguishable (chromatographic peaks close enough that order is inconsistent across experiments) — such pairs should be excluded rather than ranked

## Inputs

- Ready-to-identify candidate lists for each peak as CSV files in the 'data' folder (MS-DIAL and SIRIUS outputs)
- Trained ROASMI ensemble models (ROASMI_1–ROASMI_5) for RPLC at eluent pH ~2.7
- MS/MS scores per candidate from the identification tool (SIRIUS or MS-DIAL)

## Outputs

- Per-peak re-ranked candidate tables
- Top-k accuracy metrics (top-1, top-3, top-5) for MS/MS-only, ROASMI-alone, and ROASMI-integrated rankings
- Candidate-reduction statistic (87.3% of candidates removed when retention ordering is applied), saved to reranked_topk_results.csv

## How to apply

Load the ready-to-identify candidate lists (MS-DIAL and SIRIUS outputs) as CSV files in the 'data' folder — e.g. the 106 CASMI 2022 priority challenges and the 49 correctly-inferred-formula peaks of dataset_38. Run `python code/ROASMI_predict.py` (see PredictArgs in args.py) to compute ROASMI retention scores for every candidate, using the four-model ensemble (ROASMI_1–ROASMI_5) whose variance quantifies prediction uncertainty. Then run `python code/Identify.py` (IdentifyArgs in args.py) to integrate predicted retention scores with the MS/MS scores via the probabilistic framework of msms_rt_score_integration. Compute top-k accuracy (top-1, top-3, top-5) for the baseline MS/MS-only ranking versus the ROASMI-alone and ROASMI-integrated rankings, mirroring Fig. 6c-f, and report candidate-ranking shifts. Before integration, exclude molecular pairs that ROASMI deems retention-indistinguishable (e.g. high-variance, near-50% order probability) from downstream identification.

## Related tools

- **ROASMI** (Predicts retention-order scores for every candidate in the ready-to-identify lists via ROASMI_predict.py; ensemble variance quantifies uncertainty) — https://github.com/FangYuan717/ROASMI
- **msms_rt_score_integration** (Provides the probabilistic framework (msms_rt_score_integration) used by Identify.py to combine predicted retention scores with MS/MS scores) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **chemprop** (Backbone of ROASMI's molecular embedding module (D-MPNN) that maps candidates directly into chemical spaces tuned to retention behavior) — https://github.com/chemprop/chemprop
- **SIRIUS** (MS/MS scorer whose candidate ranking is re-ranked (average rank of the correct structure improves by ~2 units))
- **MS-DIAL** (MS/MS scorer whose candidate ranking is re-ranked (average rank of the correct structure improves by ~12 units))

## Examples

```
python code/ROASMI_predict.py && python code/Identify.py
```

## Evaluation signals

- The produced reranked_topk_results.csv must reproduce the reported 87.3% candidate-reduction figure when retention ordering is applied
- Top-k accuracy pattern must match the panels of Fig. 6c-f (top-1, top-3, top-5 for baseline, ROASMI-alone, and integrated rankings)
- Average rank of the correct structure should improve by ~2 units versus SIRIUS alone and ~12 units versus MS-DIAL alone
- Retention-score predictions should carry model-uncertainty estimates from the ensemble variance across ROASMI_1–ROASMI_5
- Pairs predicted with high order-probability variance (e.g. 9% variance) should be flagged and excluded as indistinguishable

## Limitations

- ROASMI applies only to RPLC systems (eluent pH ~2.7); constructing HILIC models with generalizability is future work
- Applies to conventional compounds within the elemental restrictions of C, H, N, O, P, and S; broader chemical space is unexplored
- The model cannot distinguish relative elution order of some close-retention pairs (e.g. 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid), and RT outliers from slight experimental condition changes remain unresolved

## Evidence

- [methods] predict step: "To predict the retention scores of each compound in the ready-to-identify datasets, run: python code/ROASMI_predict.py"
- [methods] integration step: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [abstract] compatibility finding: "ROASMI is flexible enough to be retrained with user-defined reference sets and is compatible with other MS/MS scorers"
- [discussion] integration benefit: "it seamlessly integrates with existing MS/MS tools, resulting in highly accurate annotations"
- [readme] modules description: "Designed to predict retention scores for candidate compounds during small molecule identification, ROASMI consists of two main modules."
