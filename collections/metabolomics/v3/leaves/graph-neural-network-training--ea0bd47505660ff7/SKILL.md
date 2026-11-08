---
name: graph-neural-network-training
description: Use when predicting LC retention times for metabolite annotation from
  molecular structures (InChI/SMILES), especially when transferring a pre-trained
  model to a new chromatographic method with limited labeled data.
when_to_use_negative:
- Your prediction target is not chromatographic retention time (e.g. MS/MS spectra
  or collision cross section) — the model outputs RT only
- You lack any labeled retention-time data for the target chromatographic method to
  fine-tune the Output-Block
- You need retention prediction for a technique other than liquid chromatography
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: RT-Transformer
  role: ResGAT + 1D-Transformer model trained on SMRT and fine-tuned on PredRet datasets
    to predict retention times
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:e555415c33b3dcf57d42fd340e90458036ba3da3d1d7654c5814e4dda0f91061
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# graph-neural-network-training

## Summary

Training and transfer learning of RT-Transformer, a deep neural network coupling a graph attention network (ResGAT) over molecular graphs with a 1D-Transformer over fingerprints, to predict liquid chromatography retention times under any chromatographic method.

## When to use

Use when predicting LC retention times for metabolite annotation from molecular structures (InChI/SMILES), especially when transferring a pre-trained model to a new chromatographic method with limited labeled data.

## When NOT to use

- Your prediction target is not chromatographic retention time (e.g. MS/MS spectra or collision cross section) — the model outputs RT only
- You lack any labeled retention-time data for the target chromatographic method to fine-tune the Output-Block
- You need retention prediction for a technique other than liquid chromatography

## Inputs

- CSV file named 'data.csv' with 'InChI' and 'RT' columns (molecular structures and retention times)
- Pre-trained RT-Transformer checkpoint (best_state_dict.pth or best_state_download_dict.pth)
- SMRT dataset (80,038 molecules) for pre-training or retraining
- PredRet chromatographic-method datasets for transfer learning

## Outputs

- Predicted retention times per molecule under the target chromatographic method
- Fine-tuned Output-Block weights for the new chromatographic method
- Per-dataset R2 score distributions across folds and seeds (e.g. r2_distribution.csv)

## How to apply

Prepare the dataset as a CSV with 'InChI' and 'RT' columns (SMRT pre-training set has 80,038 molecules; transfer data comes from PredRet). Combine fingerprint and molecular graph representations as model input; the architecture couples graph attention with a 1D-Transformer and an Output-Block that emits the retention time. For transfer learning, load a pre-trained checkpoint (best_state_dict.pth for full SMRT data or best_state_download_dict.pth for retained-molecule-only data), freeze the ResGAT and 1D-Transformer encoder weights, re-initialize the Output-Block, and fine-tune only that block on the target dataset, e.g. with 10-fold cross-validation repeated over multiple random seeds. Prefer the full-SMRT pre-trained model when stability matters: when transferred to 41 PredRet datasets it produced no R2 values below -0.5 and outperformed the retained-molecules model when R2 exceeded 0.75.

## Related tools

- **RT-Transformer** (ResGAT + 1D-Transformer model trained on SMRT and fine-tuned on PredRet datasets to predict retention times) — https://github.com/01dadada/RT-Transformer

## Examples

```
cp my_dataset.csv data.csv; python transfer.py
```

## Evaluation signals

- Average absolute error on the SMRT test set around 27.30 after removing not-retained molecules (33.41 with no samples removed)
- Stability criterion on the 41 PredRet transfers: no R2 value below -0.5 for the full-SMRT pre-trained model
- Full-SMRT pre-trained model outperforms the retained-molecules model whenever R2 exceeds 0.75
- Fine-tuning runs use 10-fold cross-validation repeated with five random seeds (50 runs per dataset); R2 distributions should be aggregated per dataset
- Smoke test: `python ./test.py` runs the validation/test script without errors

## Limitations

- Different chromatographic conditions yield different retention times for the same metabolite, so a model trained on one method may not transfer without fine-tuning
- Two pre-trained checkpoints exist (full SMRT vs retained molecules only); choosing the wrong one affects stability — the retained-molecules model was less stable on the 41 PredRet datasets
- No changelog is available for the 01dadada/RT-Transformer repository, so code evolution cannot be tracked

## Evidence

- [abstract] a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [results] the model pre-trained on the complete SMRT dataset was more stable, with no R2 scores below -0.5, and outperformed the retained-molecules model when R2 exceeded 0.75: [evidence span withheld — anchor status: not_found]
- [readme] Prepare your dataset as a csv file which has "InChI" and "RT" columns. Rename it as "data.csv" at the root directory. Run transfer.py: "Prepare your dataset as a csv file which has "InChI" and "RT" columns.
- Rename it as "data.csv" at the root directory.
- Run transfer.py"
- [readme] The SMRT dataset is collect from this paper; Datasets for transfer learning is download from PredRet: "The SMRT dataset is collect from [this paper](https://doi.org/10.1038/s41467-019-13680-7)
Datasets for transfer learning is download from [PredRet](http://predret.org/)"
- [abstract] the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
