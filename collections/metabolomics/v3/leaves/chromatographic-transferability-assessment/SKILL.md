---
name: chromatographic-transferability-assessment
description: Use when deciding whether fine-tuning a pre-trained ROASMI model with
  a specific reference dataset will yield substantial rank-score gains on a target
  prediction set, or when grouping retraining results by ChemSS/ChrSS similarity.
when_to_use_negative:
- The prediction set lies in a chemical space outside the elemental restrictions of
  C, H, N, O, P, S that ROASMI was built for, or in a non-RPLC chromatographic space
  (e.g., HILIC), since the initial model is limited to RPLC at eluent pH ≈ 2.7.
- Two compounds' chromatographic peaks are close enough that their retention order
  may not be consistent across experiments — such pairs should be excluded from identification
  rather than assessed for transferability.
- The goal is de novo RT prediction (retention-time regression) rather than retention-order
  ranking, since ROASMI predicts ordinal scores, not absolute RTs.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: ROASMI
  role: Retention-order model fine-tuned with user-defined reference sets; its rank-score
    deltas are the transferability metric.
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: D-MPNN molecular embedding module extended by ROASMI; provides the representation
    that is fine-tuned during retraining.
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Underlying deep-learning framework for the D-MPNN and RankNet modules.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:43eeec6dacadce41bb5099e3ed92a0c653f51d67de5ae6cd208bda64d4f55c4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chromatographic-transferability-assessment

## Summary

Assess whether a retrained ROASMI retention-order model transfers to a new dataset by quantifying rank-score improvements as a function of chemical-space similarity (ChemSS) and chromatographic-space similarity (ChrSS) between the retraining reference and prediction sets.

## When to use

Use when deciding whether fine-tuning a pre-trained ROASMI model with a specific reference dataset will yield substantial rank-score gains on a target prediction set, or when grouping retraining results by ChemSS/ChrSS similarity.

## When NOT to use

- The prediction set lies in a chemical space outside the elemental restrictions of C, H, N, O, P, S that ROASMI was built for, or in a non-RPLC chromatographic space (e.g., HILIC), since the initial model is limited to RPLC at eluent pH ≈ 2.7.
- Two compounds' chromatographic peaks are close enough that their retention order may not be consistent across experiments — such pairs should be excluded from identification rather than assessed for transferability.
- The goal is de novo RT prediction (retention-time regression) rather than retention-order ranking, since ROASMI predicts ordinal scores, not absolute RTs.

## Inputs

- Initial pre-trained ROASMI model checkpoints (ROASMI_1–ROASMI_5)
- Reference-set dataset for fine-tuning (e.g., dataset_83 as reference, dataset_84 as test for HILIC; 12 RPLC reference/prediction combinations A1–A12)
- Prediction-set dataset with compounds requiring retention scores
- ChemSS and ChrSS similarity values for each reference/prediction pair

## Outputs

- Rank scores for each compound pair under the initial-train and retrained models
- Rank-score delta per compound pair
- Transferability summary table grouping deltas by ChemSS and ChrSS
- Dataset-size ablation results (rank-score delta vs. number of reference compounds)
- Reproduction of Supplementary Fig. 4 patterns

## How to apply

Fine-tune the D-MPNN molecular embedding module and RankNet retention prediction module with a chosen reference set by running `python code/ROASMI_train.py`, then predict retention scores for the paired prediction set via `python code/ROASMI_predict.py` using the retrained model. Compute rank-score deltas for each compound pair between the initial-train and retrained predictions, then group the twelve reference/prediction dataset combinations by their chemical-space similarity (ChemSS) and chromatographic-space similarity (ChrSS). Substantial improvements only occur when the reference set shares overlapping chemical and chromatographic spaces with the prediction set (e.g., A1 and A2 gained 0.21 and 0.1, while A11 and A12 showed negligible gains), so the similarity grouping is the transferability signal. As an ablation, vary the number of reference compounds per retraining run to check whether rank-score deltas depend on dataset size. Reproduce the reported pattern in Supplementary Fig. 4 and record all rank scores, deltas, and similarity metrics in one summary table.

## Related tools

- **ROASMI** (Retention-order model fine-tuned with user-defined reference sets; its rank-score deltas are the transferability metric.) — https://github.com/FangYuan717/ROASMI
- **chemprop** (D-MPNN molecular embedding module extended by ROASMI; provides the representation that is fine-tuned during retraining.) — https://github.com/chemprop/chemprop
- **PyTorch** (Underlying deep-learning framework for the D-MPNN and RankNet modules.)

## Examples

```
python code/ROASMI_train.py && python code/ROASMI_predict.py
```

## Evaluation signals

- Rank-score deltas for high-similarity combinations (e.g., A1 ≈ 0.21, A2 ≈ 0.1) should exceed those for low-similarity combinations (e.g., A11, A12 near zero).
- Grouped transferability trends by ChemSS and ChrSS must match Supplementary Fig. 4.
- Retrained model predictions must come from the same ROASMI_train.py / ROASMI_predict.py pipeline used for the initial-train model, enabling direct delta computation.
- Dataset-size ablation: rank-score deltas should be reported across varied reference-set sizes, demonstrating the effect of training-set size on transferability.
- Uncertainty can be quantified via the variance of retention-order predictions across the ensemble (ROASMI_1–ROASMI_5).

## Limitations

- ROASMI is limited to conventional compounds within elemental restrictions of C, H, N, O, P, and S; broader chemical space is unexplored.
- ROASMI currently applies only to RPLC systems; constructing generalizable HILIC retention models remains future work (a HILIC example using dataset_83 as reference and dataset_84 as test is provided for retraining demonstration).
- The model cannot distinguish certain molecular pairs with very close retention (e.g., 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid), and such pairs must be excluded from identification.
- RT outliers from slight experimental condition changes are unresolved and require in-depth mechanistic study.
- Fine-tuning gains depend on overlapping chemical and chromatographic spaces; a reference set from a dissimilar space yields negligible improvement (A11, A12).

## Evidence

- [results] Fine-tuning ROASMI yields substantial rank-score improvements only when the reference set shares overlapping chemical and chromatographic spaces with the prediction set.: [evidence span withheld — anchor status: not_found]
- [methods] Retraining procedure: "To retrain ROASMI, run: python code/ROASMI_train.py"
- [methods] Prediction procedure: "To predict the retention scores of each compound in the ready-to-identify datasets, run: python code/ROASMI_predict.py"
- [discussion] RPLC-only scope: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
- [discussion] Indistinguishable pairs: "Such pairs should be excluded from the subsequent small molecule identification process."
- [readme] Ensemble uncertainty: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [methods] HILIC retraining example: "retrain two HILIC models using 'dataset_83' as the reference set and 'dataset_84' as the test set"
