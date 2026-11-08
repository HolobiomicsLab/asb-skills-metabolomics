---
name: transformer-sequence-regression
description: Use when the goal is to predict retention times for small molecules under
  a given chromatographic method from InChI/graph inputs, or to transfer a pre-trained
  RT-Transformer to a new chromatographic condition and benchmark its accuracy.
when_to_use_negative:
- The task requires predicting retention times under multiple different chromatographic
  conditions from a single model without transfer learning — the article notes current
  methods 'lack sufficient scalability to transfer from one specific chromatographic
  method to another', and this skill's transfer path requires per-condition fine-tuning.
- The molecules of interest are not retained under the chromatographic method (RT
  ≤ 300 s) and the retained-data evaluation protocol is required — in that case the
  full-data checkpoint (33.41 s MAE) or a different evaluation condition is more appropriate.
- Input data lacks 'InChI' and 'RT' columns in the expected CSV format and cannot
  be converted, e.g. raw chromatographic peak tables without molecular identifiers.
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3520
tools:
- name: RT-Transformer
  role: Deep neural network coupling a graph attention network with a 1D-Transformer
    that is trained or fine-tuned on SMRT data to predict retention times; provides
    train.py, transfer.py, test.py, and QuickStart.ipynb entry points.
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:d2900bc9d0a14e93b487c3e921d0f3d2422e680794adc63165a5a9a19ea6b265
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transformer-sequence-regression

## Summary

Trains and fine-tunes RT-Transformer, a deep neural network coupling a graph attention network with a 1D-Transformer, to predict liquid chromatography retention times of metabolites and evaluate MAE/MedAE on SMRT-derived test partitions.

## When to use

Use when the goal is to predict retention times for small molecules under a given chromatographic method from InChI/graph inputs, or to transfer a pre-trained RT-Transformer to a new chromatographic condition and benchmark its accuracy.

## When NOT to use

- The task requires predicting retention times under multiple different chromatographic conditions from a single model without transfer learning — the article notes current methods 'lack sufficient scalability to transfer from one specific chromatographic method to another', and this skill's transfer path requires per-condition fine-tuning.
- The molecules of interest are not retained under the chromatographic method (RT ≤ 300 s) and the retained-data evaluation protocol is required — in that case the full-data checkpoint (33.41 s MAE) or a different evaluation condition is more appropriate.
- Input data lacks 'InChI' and 'RT' columns in the expected CSV format and cannot be converted, e.g. raw chromatographic peak tables without molecular identifiers.

## Inputs

- CSV file with 'InChI' and 'RT' columns (renamed to data.csv at the repository root)
- SMRT dataset with 80 038 molecules, optionally filtered to retention times > 300 s (retained molecules)
- Pre-trained model checkpoint (best_state_dict.pth or best_state_download_dict.pth)
- Molecular structures of query metabolites (InChI strings) for prediction

## Outputs

- Predicted retention times for query molecules
- MAE and MedAE metrics on the SMRT test partition (target: MAE 27.30 s, MedAE 12.46 s for retained data; 33.41 s MAE for full data)
- Fine-tuned RT-Transformer model checkpoint for a new chromatographic condition

## How to apply

Prepare each dataset as a CSV with 'InChI' and 'RT' columns; the RT-Transformer couples molecular graph features (fed to a graph attention network) with molecular sequence/fingerprint features (fed to a 1D-Transformer) to regress retention time. For training from scratch on the 80 038-molecule SMRT dataset, first filter out not-retained molecules (RT > 300 s) to match the retained-data evaluation condition, rename the filtered file to 'data.csv' at the repository root, and run train.py; for evaluation or transfer, download 'best_state_dict.pth' (full-data model) or 'best_state_download_dict.pth' (retained-data model) from the Hugging Face space, rename the target dataset to 'data.csv', and run transfer.py or test.py. Predict retention times for new molecules via the QuickStart.ipynb notebook or the Hugging Face web server. Judge correctness by comparing computed MAE and MedAE on the test partition against reported values: MAE 27.30 s and MedAE 12.46 s for the retained-molecules condition (33.41 s MAE when no samples are removed).

## Related tools

- **RT-Transformer** (Deep neural network coupling a graph attention network with a 1D-Transformer that is trained or fine-tuned on SMRT data to predict retention times; provides train.py, transfer.py, test.py, and QuickStart.ipynb entry points.) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./train.py  # after renaming the filtered SMRT subset to data.csv (InChI, RT columns)
```

## Evaluation signals

- Computed MAE on the SMRT test partition for the retained-molecules (RT > 300 s) condition matches 27.30 s and MedAE matches 12.46 s.
- Computed MAE on the full SMRT test set (no samples removed) matches 33.41 s.
- Input CSV has the required 'InChI' and 'RT' columns and is renamed to 'data.csv' at the repository root before running train.py or transfer.py.
- The correct pre-trained checkpoint is used: best_state_download_dict.pth for retained-data models, best_state_dict.pth for full-data models.
- Predicted retention times are non-negative and within a plausible chromatographic range comparable to the training RT distribution.

## Limitations

- The retained-molecules evaluation protocol removes molecules with RT ≤ 300 s, which changes the error baseline (27.30 s vs 33.41 s MAE); results are only comparable under the same filtering condition.
- No changelog was found for the repository 01dadada/RT-Transformer, so code versions and reproducibility state are not fully traceable.
- Model performance depends on the chromatographic condition; the article notes that different conditions 'may result in different retention times for the same metabolite', so a model trained under one condition should be transferred rather than used directly on another.

## Evidence

- [abstract] RT-Transformer is a deep neural network coupling graph attention network with 1D-Transformer for retention time prediction: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] MAE 27.30 after removing not-retained molecules vs 33.41 on full data: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [abstract] Pre-training on 80 038-molecule SMRT dataset then transfer to different chromatographic methods: "First, we obtain a pre-trained model by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules, and then transfer the resulting model to different"
- [readme] Data format requirement: CSV with InChI and RT columns renamed to data.csv: "Prepare your dataset as a csv file which has "InChI" and "RT" columns.
- Rename it as "data.csv" at the root directory.
- Run train.py"
- [readme] Pre-trained checkpoint naming distinguishes retained vs full data models: "best_state_download_dict.pth The Best model of RT-Transformer train from retained data.
best_state_dict.pth The Best model of RT-Transformer train from full data."
