---
name: casmi-challenge-candidate-ranking
description: Use when you have ready-to-identify candidate lists (e.g. SIRIUS or MS-DIAL
  outputs for CASMI 2022 challenges) and want to check whether retention-order scores
  improve correct-structure ranking or prune candidates before MS/MS scoring.
when_to_use_negative:
- The chromatography is HILIC rather than reversed-phase LC — ROASMI currently applies
  only to RPLC systems and the shipped models assume eluent pH around 2.7
- Candidate structures fall outside the elemental restrictions of C, H, N, O, P, and
  S, where ROASMI's chemical-space coverage is unexplored
- The goal is de novo structure elucidation without a pre-computed candidate list
  — this skill only re-ranks existing ready-to-identify candidate sets
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: ROASMI
  role: Predicts retention-order scores for every candidate in each ready-to-identify
    list via its molecular embedding (D-MPNN/chemprop) and RankNet retention modules
  repo: https://github.com/FangYuan717/ROASMI
- name: msms_rt_score_integration
  role: Probabilistic framework used by Identify.py to integrate predicted retention
    scores with MS/MS scores for the integrated re-ranking
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: chemprop
  role: Extended by ROASMI as the molecular embedding module; D-MPNN learns representations
    directly from compound structure
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning backend for ROASMI training and prediction (GPU-enabled install
    optional)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:1c656a947b4eb2d42486b6c1665a2781e11c5e8fb4d15d0e9b987f44b1460e6a
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# casmi-challenge-candidate-ranking

## Summary

Re-ranks small-molecule candidate lists from MS/MS tools (SIRIUS, MS-DIAL) using ROASMI-predicted retention scores alone and integrated with MS/MS scores via the msms_rt_score_integration probabilistic framework, then evaluates top-k accuracy and candidate reduction on the CASMI 2022 and dataset_38 re-annotation benchmarks.

## When to use

Use when you have ready-to-identify candidate lists (e.g. SIRIUS or MS-DIAL outputs for CASMI 2022 challenges) and want to check whether retention-order scores improve correct-structure ranking or prune candidates before MS/MS scoring.

## When NOT to use

- The chromatography is HILIC rather than reversed-phase LC — ROASMI currently applies only to RPLC systems and the shipped models assume eluent pH around 2.7
- Candidate structures fall outside the elemental restrictions of C, H, N, O, P, and S, where ROASMI's chemical-space coverage is unexplored
- The goal is de novo structure elucidation without a pre-computed candidate list — this skill only re-ranks existing ready-to-identify candidate sets

## Inputs

- Ready-to-identify candidate lists as CSVs from MS-DIAL and SIRIUS (data folder) for the 106 CASMI 2022 priority challenges and 49 dataset_38 peaks
- Trained ROASMI models (ROASMI_1–ROASMI_5) for RPLC retention-order prediction at eluent pH ≈ 2.7
- identify_toy_set.csv reference input ('data/identify_toy_set.csv') for pipeline validation

## Outputs

- reranked_topk_results.csv with per-peak re-ranked candidate tables
- Top-1 / top-3 / top-5 accuracy summaries for MS/MS-only, ROASMI-alone, and ROASMI-integrated rankings
- Candidate-reduction statistic (target: 87.3% of candidates removed by retention ordering)

## How to apply

Load the ready-to-identify candidate lists as CSV files in the 'data' folder — 106 CASMI 2022 priority challenges and 49 correctly-inferred-formula peaks of dataset_38. Run `python code/ROASMI_predict.py` (see PredictArgs in args.py) to compute ROASMI retention scores for every candidate, using the ensemble of four/five initial models (ROASMI_1–ROASMI_5, RPLC eluent pH ≈ 2.7) whose prediction variance quantifies uncertainty. Re-rank candidates by retention score alone, then run `python code/Identify.py` (IdentifyArgs) to integrate retention scores with MS/MS scores through msms_rt_score_integration's probabilistic framework. Compute top-1, top-3, and top-5 accuracy for the baseline MS/MS-only ranking versus ROASMI-alone and ROASMI-integrated rankings, mirroring Fig. 6c-f, and record per-peak rank shifts (expected average improvement: 2 rank units vs SIRIUS alone, 12 vs MS-DIAL alone). Finally, apply the retention ordering as a candidate filter and verify the reported 87.3% candidate-reduction statistic (Supplementary Table 3), saving per-peak re-ranked tables and summary metrics to reranked_topk_results.csv.

## Related tools

- **ROASMI** (Predicts retention-order scores for every candidate in each ready-to-identify list via its molecular embedding (D-MPNN/chemprop) and RankNet retention modules) — https://github.com/FangYuan717/ROASMI
- **msms_rt_score_integration** (Probabilistic framework used by Identify.py to integrate predicted retention scores with MS/MS scores for the integrated re-ranking) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **chemprop** (Extended by ROASMI as the molecular embedding module; D-MPNN learns representations directly from compound structure) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning backend for ROASMI training and prediction (GPU-enabled install optional))

## Examples

```
python code/ROASMI_predict.py  # compute retention scores for every candidate, then: python code/Identify.py
```

## Evaluation signals

- The output must reproduce the reported 87.3% candidate-reduction figure when retention ordering is applied as a filter
- Top-k accuracy pattern (top-1/3/5) for the integrated ranking must match the panels of Fig. 6c-f
- Average rank of the correct structure should shift by about 2 units versus SIRIUS alone and 12 units versus MS-DIAL alone
- Ranking shifts on the toy input 'identify_toy_set.csv' should run cleanly end-to-end before applying to the 106 CASMI challenges
- Ensemble variance across ROASMI_1–ROASMI_5 should flag indistinguishable pairs; such pairs should be excluded from subsequent identification

## Limitations

- Indistinguishable molecular pairs with close retention (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid) cannot be ordered and should be excluded from identification
- When chromatographic peaks are close enough, retention order may not be consistent across experiments, and RT outliers from slight condition changes remain unresolved
- Machine-learning retention predictions are imperfect and should be integrated with mechanistic understanding rather than trusted blindly

## Evidence

- [results] Re-ranking with ROASMI improved correct-structure rank: [evidence span withheld — anchor status: not_found]
- [methods] Integration command in the workflow: [evidence span withheld — anchor status: not_found]
- [readme] RankNet retention module: "The retention prediction module.** A Ranking Neural Network (RankNet) was used"
- [other] Candidate reduction statistic: [evidence span withheld — anchor status: not_found]
- [discussion] Scope limit to RPLC: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
