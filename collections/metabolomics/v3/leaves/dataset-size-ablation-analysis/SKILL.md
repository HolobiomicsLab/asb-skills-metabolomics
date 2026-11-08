---
name: dataset-size-ablation-analysis
description: Use when fine-tuning a pre-trained ROASMI model with a retraining reference
  dataset and you want to quantify how many reference compounds are needed for transferability,
  grouping results by ChemSS and ChrSS.
when_to_use_negative:
- The prediction set is outside the RPLC system at eluent pH ≈ 2.7, e.g. HILIC retention
  modeling, which the paper leaves as future work
- The prediction set shares no overlapping chemical or chromatographic space with
  the retraining reference set, where rank-score gains are negligible
- The goal is absolute RT prediction or identification of compounds outside C/H/N/O/P/S
  elemental restrictions rather than retention-order transferability
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ROASMI
  role: Pre-trained retention-order model fine-tuned on reference datasets of varying
    size; ROASMI_train.py and ROASMI_predict.py drive the retraining and prediction
    steps
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Provides the D-MPNN molecular embedding module that is fine-tuned during each
    retraining run
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Backend used to fine-tune the embedding and RankNet retention prediction modules
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:82d0fea0e8118ea1283b6c40c9dfd745fa733dfadaf15a91de78a238a3d4a2d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Dataset-size ablation analysis

## Summary

Fine-tune a pre-trained ROASMI model on reference sets of varying size (number of reference compounds) and record how rank-score deltas change with dataset size, as described for the 12 retraining combinations A1–A12.

## When to use

Use when fine-tuning a pre-trained ROASMI model with a retraining reference dataset and you want to quantify how many reference compounds are needed for transferability, grouping results by ChemSS and ChrSS.

## When NOT to use

- The prediction set is outside the RPLC system at eluent pH ≈ 2.7, e.g. HILIC retention modeling, which the paper leaves as future work
- The prediction set shares no overlapping chemical or chromatographic space with the retraining reference set, where rank-score gains are negligible
- The goal is absolute RT prediction or identification of compounds outside C/H/N/O/P/S elemental restrictions rather than retention-order transferability

## Inputs

- Initial pre-trained ROASMI model checkpoints (ROASMI_1–ROASMI_5, ensemble for RPLC at eluent pH ≈ 2.7)
- 12 retraining reference-set / prediction-set dataset combinations (e.g., dataset_83 as reference and dataset_84 as test set for HILIC models)
- SMILES/CID annotation tables for the prediction-set compounds (e.g., predict_toy_set.csv, identify_toy_set.csv)
- ChemSS and ChrSS similarity values for each reference–prediction-set pair

## Outputs

- Per-combination rank scores from initial-train and retrained models and their rank-score deltas for each compound pair
- Transferability summary table with ChemSS/ChrSS groupings and dataset-size ablation results

## How to apply

Load the initial pre-trained ROASMI checkpoints (ROASMI_1–ROASMI_5) together with the 12 retraining/prediction-set combinations from the packaged data folder. For each combination, fine-tune the D-MPNN molecular embedding module and the RankNet retention prediction module via code/ROASMI_train.py using the reference set as training data, varying the number of reference compounds per run to build a size ablation. Predict retention scores for the paired prediction-set compounds with code/ROASMI_predict.py and compute rank-score deltas between the initial-train and retrained predictions for each compound pair. Group datasets by chemical-space similarity (ChemSS) and chromatographic-space similarity (ChrSS) to see that substantial improvements (e.g., A1 +0.21, A2 +0.1) only occur when reference and prediction sets share overlapping chemical and chromatographic spaces, whereas low-ChemSS combinations (e.g., A11, A12) show negligible gains. Assemble rank scores, deltas, transferability metrics and ablation results into a single summary table and check the ChemSS/ChrSS trends against Supplementary Fig. 4.

## Related tools

- **ROASMI** (Pre-trained retention-order model fine-tuned on reference datasets of varying size; ROASMI_train.py and ROASMI_predict.py drive the retraining and prediction steps) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Provides the D-MPNN molecular embedding module that is fine-tuned during each retraining run) — https://github.com/chemprop/chemprop
- **PyTorch** (Backend used to fine-tune the embedding and RankNet retention prediction modules)

## Examples

```
python code/ROASMI_train.py && python code/ROASMI_predict.py
```

## Evaluation signals

- For each of the 12 combinations the summary table reproduces the reported initial-train, retrained and delta rank-score patterns
- High-ChemSS/ChrSS combinations (A1, A2) show large positive deltas (0.21 and 0.1) while low-similarity combinations (A11, A12) show negligible gains
- ChemSS/ChrSS transferability trends match Supplementary Fig. 4
- Increasing the number of reference compounds monotonically shifts the rank-score deltas in the expected direction for each ablation run
- Retrained predictions for the prediction-set compounds are produced for every combination without missing values

## Limitations

- ROASMI currently applies only to RPLC systems; HILIC models with generalizability are future work
- Retention order of very close chromatographic peaks may not be consistent across experiments, and pairs such as 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid are indistinguishable
- Model coverage is restricted to compounds within the elemental restrictions of C, H, N, O, P and S
- RT outliers from slight experimental condition changes require in-depth mechanistic studies and are not resolved by the ablation

## Evidence

- [abstract] substantial improvements only when reference and prediction sets share overlapping chemical and chromatographic spaces: [evidence span withheld — anchor status: not_found]
- [methods] fine-tune the D-MPNN embedding and RankNet modules with reference sets: "To retrain ROASMI, run: python code/ROASMI_train.py"
- [methods] rank-score deltas from retrained vs initial predictions: [evidence span withheld — anchor status: not_found]
- [intro] transferability by grouping datasets by ChemSS and ChrSS: "determining the degree of ChrSS required for replicable retention orders remains to be explored"
- [discussion] RPLC-only scope limits ablation transferability: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
