---
name: chromatographic-rt-dataset-preparation
description: Use when you need to train or retrain RT-Transformer on a retention time
  dataset prepared as InChI+RT CSV, or reproduce the SMRT full-data MAE of 33.41 s.
when_to_use_negative:
- Your input is already a processed prediction (per-molecule RT values) and you only
  need to score MAE/MedAE on an existing test set without training — test.py covers
  validation only.
- Your data lacks InChI structures or RT annotations, e.g. only SMILES strings without
  retention time labels — the pipeline requires 'InChI' and 'RT' columns.
- You need retention time prediction under a single, well-covered chromatographic
  method where the pretrained model already suffices; then use the Hugging Face web
  server or QuickStart.ipynb instead of retraining.
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3391
tools:
- name: RT-Transformer
  role: The deep learning model (graph attention network coupled with 1D-Transformer)
    trained on the prepared dataset to predict retention times.
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:9b683605cd19962bbc2e945f777946819d61738663836cd81db17efa176f2771
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# chromatographic-rt-dataset-preparation

## Summary

Prepares a chromatographic retention time dataset (e.g. the SMRT small-molecule dataset with 80 038 molecules or a transfer-learning set from PredRet) as a CSV with InChI and RT columns, and trains the RT-Transformer (graph attention network + 1D-Transformer) on it, evaluating MAE/MedAE on the test split.

## When to use

Use when you need to train or retrain RT-Transformer on a retention time dataset prepared as InChI+RT CSV, or reproduce the SMRT full-data MAE of 33.41 s.

## When NOT to use

- Your input is already a processed prediction (per-molecule RT values) and you only need to score MAE/MedAE on an existing test set without training — test.py covers validation only.
- Your data lacks InChI structures or RT annotations, e.g. only SMILES strings without retention time labels — the pipeline requires 'InChI' and 'RT' columns.
- You need retention time prediction under a single, well-covered chromatographic method where the pretrained model already suffices; then use the Hugging Face web server or QuickStart.ipynb instead of retraining.

## Inputs

- CSV with 'InChI' and 'RT' columns (retention time in seconds), renamed data.csv at repository root
- SMRT small molecule retention time dataset (80 038 molecules)
- PredRet datasets for the 5 chromatographic-condition transfer-learning datasets
- Pretrained model files best_state_dict.pth / best_state_download_dict.pth from the Hugging Face space

## Outputs

- Trained RT-Transformer model checkpoint (best_state_dict.pth or best_state_download_dict.pth)
- Test-set retention time predictions in seconds
- Summary metrics: mean absolute error and median absolute error (e.g. MAE 33.41 s, MedAE 13.05 s for the full-data SMRT condition)

## How to apply

Assemble the dataset as a CSV with 'InChI' and 'RT' columns and rename it data.csv at the repository root; SMRT (80 038 molecules, from the 10.1038/s41467-019-13680-7 paper) supports pre-training, PredRet datasets support transfer learning. Decide whether to keep all molecules (including not-retained ones) or remove not-retained molecules — this choice maps to the two pretrained checkpoints best_state_dict.pth (full data) and best_state_download_dict.pth (retained data), and to the reported MAE of 33.41 s vs 27.30 s. Train with train.py (retraining) or transfer.py after downloading the pretrained model from the Hugging Face space. Evaluate on the test split to obtain MAE and MedAE in seconds; a correct full-data run reproduces MAE 33.41 s and MedAE 13.05 s.

## Related tools

- **RT-Transformer** (The deep learning model (graph attention network coupled with 1D-Transformer) trained on the prepared dataset to predict retention times.) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./train.py  # data.csv with InChI and RT columns prepared at repo root; for transfer learning: python transfer.py
```

## Evaluation signals

- Reproducing the SMRT full-data (no samples removed) test-set MAE of 33.41 s and MedAE 13.05 s
- Removing not-retained molecules should drop the average absolute error to 27.30 s, matching the reported figure
- Dataset file must be data.csv with 'InChI' and 'RT' columns before running train.py or transfer.py
- Using the correct checkpoint for the chosen condition: best_state_dict.pth (full data) vs best_state_download_dict.pth (retained data)
- Per-molecule predictions and summary metrics should be exported (e.g. rt_transformer_all_molecules_predictions.csv) for reproducibility

## Limitations

- Different chromatographic conditions yield different retention times for the same metabolite; a model trained on one method may not transfer without fine-tuning on a method-specific dataset.
- The reported MAE depends strongly on whether not-retained molecules are removed (27.30 vs 33.41 s), so the condition must be stated explicitly.
- Requirements include Python 3.9, torch, torch-geometric, torch-scatter, torch-sparse, torch-cluster and rdkit-pypi; missing these will break graph/sequence preprocessing.

## Evidence

- [readme] Prepare your dataset as a csv file which has "InChI" and "RT" columns.: [evidence span withheld — anchor status: external_doc]
- [readme] The RT-Tranformer combine the fingerprint and the molecular graph data and predict retention time as the output.: [evidence span withheld — anchor status: external_doc]
- [abstract] the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] The SMRT dataset is collect from [this paper](https://doi.org/10.1038/s41467-019-13680-7): [evidence span withheld — anchor status: external_doc]
- [readme] Datasets for transfer learning is download from [PredRet](http://predret.org/): [evidence span withheld — anchor status: external_doc]
