---
name: topk-accuracy-evaluation
description: Use when a candidate-ranking or annotation method must be compared against
  an MS/MS-only baseline (e.g., SIRIUS or MS-DIAL) to quantify whether integrating
  an additional scorer (e.g., retention scores) improves identification accuracy.
when_to_use_negative:
- When the chromatographic system is HILIC rather than RPLC — ROASMI retention scores
  that feed this evaluation currently apply only to RPLC systems
- When candidates fall outside the elemental restrictions of C, H, N, O, P, and S
  covered by the ROASMI models
- When candidate pairs have retention peaks so close that the retention order is inconsistent
  across experiments (indistinguishable pairs should be excluded from identification)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ROASMI
  role: Supplies the predicted retention scores used to re-rank candidates alongside
    the MS/MS-only baseline
  repo: https://github.com/FangYuan717/ROASMI
- name: msms_rt_score_integration
  role: Probabilistic framework used to integrate predicted retention scores with
    MS/MS scores before computing top-k accuracy
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: pandas
  role: Used to load candidate CSVs and compute/aggregate top-k accuracy and candidate-reduction
    statistics
- name: SIRIUS
  role: Source of the MS/MS-only candidate ranking baseline (average rank of correct
    structure improves by 2 units when ROASMI is added)
- name: MS-DIAL
  role: Source of the second MS/MS-only candidate ranking baseline (average rank of
    correct structure improves by 12 units when ROASMI is added)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:99bc093d61e9de56a3f1d43c654aab305340b48555173fa9f3c7e83380be62a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - MS-DIAL
  - PyTorch
  - ROASMI
  - SIRIUS
  - msms_rt_score_integration
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# topk-accuracy-evaluation

## Summary

Benchmark protocol for evaluating candidate-ranking improvements in small-molecule identification by computing top-k accuracy (top-1, top-3, top-5) for MS/MS-only baselines versus retention-score-integrated rankings, as used to validate ROASMI on CASMI 2022 and dataset_38.

## When to use

Use when a candidate-ranking or annotation method must be compared against an MS/MS-only baseline (e.g., SIRIUS or MS-DIAL) to quantify whether integrating an additional scorer (e.g., retention scores) improves identification accuracy.

## When NOT to use

- When the chromatographic system is HILIC rather than RPLC — ROASMI retention scores that feed this evaluation currently apply only to RPLC systems
- When candidates fall outside the elemental restrictions of C, H, N, O, P, and S covered by the ROASMI models
- When candidate pairs have retention peaks so close that the retention order is inconsistent across experiments (indistinguishable pairs should be excluded from identification)

## Inputs

- Ready-to-identify candidate lists as CSV files (MS-DIAL and SIRIUS outputs) for 106 CASMI 2022 priority challenges
- Candidate lists for the 49 correctly-inferred-formula peaks of dataset_38
- Per-candidate MS/MS scores and ROASMI retention scores (from ROASMI_predict.py output)

## Outputs

- Per-peak re-ranked candidate tables
- Top-k accuracy (top-1, top-3, top-5) per ranking scheme: MS/MS-only baseline, retention-score-alone, and integrated
- Candidate-reduction statistic (expect 87.3%)
- Summary metrics saved to reranked_topk_results.csv

## How to apply

Load ready-to-identify candidate lists (MS-DIAL and SIRIUS outputs as CSV files in the 'data' folder) for the 106 CASMI 2022 priority challenges and the 49 correctly-inferred-formula peaks of dataset_38. For each ranking scheme — baseline MS/MS-only, retention-score-alone, and integrated (msms_rt_score_integration probabilistic framework) — compute top-k accuracy at k = 1, 3, and 5, mirroring the panels of Fig. 6c-f. Summarize rank shifts of the correct structure (the paper reports +2 units improvement over SIRIUS alone and +12 units over MS-DIAL alone) and the candidate-reduction statistic (87.3% of candidates removed by retention ordering, per Supplementary Table 3). Write per-peak re-ranked candidate tables and summary accuracy/candidate-reduction metrics to reranked_topk_results.csv, and treat reproduction of the 87.3% figure and the Fig. 6c-f accuracy pattern as the validation gate.

## Related tools

- **ROASMI** (Supplies the predicted retention scores used to re-rank candidates alongside the MS/MS-only baseline) — https://github.com/FangYuan717/ROASMI
- **msms_rt_score_integration** (Probabilistic framework used to integrate predicted retention scores with MS/MS scores before computing top-k accuracy) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **pandas** (Used to load candidate CSVs and compute/aggregate top-k accuracy and candidate-reduction statistics)
- **SIRIUS** (Source of the MS/MS-only candidate ranking baseline (average rank of correct structure improves by 2 units when ROASMI is added))
- **MS-DIAL** (Source of the second MS/MS-only candidate ranking baseline (average rank of correct structure improves by 12 units when ROASMI is added))

## Examples

```
python code/Identify.py  # integrate ROASMI retention scores with MS/MS scores, then compute top-1/3/5 accuracy into reranked_topk_results.csv
```

## Evaluation signals

- The computed candidate-reduction statistic must reproduce the reported 87.3% when retention ordering is applied (Supplementary Table 3)
- The top-k accuracy pattern must match the panels of Fig. 6c-f (integrated ranking ≥ retention-alone ≥ MS/MS-only baseline)
- Average rank of the correct structure should improve by ~2 units versus SIRIUS alone and ~12 units versus MS-DIAL alone
- Per-peak re-ranked tables in reranked_topk_results.csv must retain all original candidates with their MS/MS and retention scores before re-ranking
- Ranking shifts should be reviewed to confirm they match the paper's reported values

## Limitations

- Evaluation is only meaningful on benchmarks where the ground-truth structure is known for each peak (e.g., CASMI 2022 challenges, correctly-inferred-formula peaks of dataset_38)
- Retention-score re-ranking cannot distinguish candidate pairs whose chromatographic peaks are so close that ROASMI reports them as indistinguishable (e.g., 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid)
- Results depend on the reference-set-trained ROASMI models; scores are specific to the RPLC system with eluent pH around 2.7
- RT outliers from slight experimental condition changes can degrade ranking fidelity

## Evidence

- [methods] top-k accuracy computation: [evidence span withheld — anchor status: not_found]
- [methods] candidate reduction: [evidence span withheld — anchor status: not_found]
- [methods] integration step: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [abstract] integration rationale: "ROASMI is flexible enough to be retrained with user-defined reference sets and is compatible with other MS/MS scorers"
- [readme] indistinguishable pairs: "when the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments."
