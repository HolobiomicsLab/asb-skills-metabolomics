---
name: retention-time-prediction-fine-tuning
description: Use when predicting liquid chromatography retention times for a new chromatographic
  method from molecular structures (InChI/SMILES), especially with limited labeled
  data where training a retention-time model from scratch is infeasible.
when_to_use_negative:
- When no pre-trained checkpoint applies and you have a large labeled dataset for
  the target method — retrain from scratch with train.py instead
- When the input lacks structural identifiers (InChI/SMILES) convertible to molecular
  graphs and fingerprints — the model requires graph and fingerprint inputs, not tabular
  metadata
- When the target task is not liquid chromatography retention time prediction (e.g.
  mass spectra prediction or general property regression)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: RT-Transformer
  role: Deep neural network (graph attention network + 1D-Transformer) used both for
    pre-training on SMRT and for transfer-learning fine-tuning to new chromatographic
    methods
  repo: https://github.com/01dadada/RT-Transformer
- name: RT-Transformer web server
  role: Hosts the pre-trained model files for download and provides a browser interface
    for predicting retention times on user data
  repo: https://huggingface.co/spaces/Xue-Jun/RT-Transformer
- name: PredRet
  role: Source of the 41 external retention time datasets used as transfer-learning
    targets in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:371e4e88ddc238b33dbf62f44a09ed8f1d0f0838465461d535aa61ce09d0ed16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - PredRet
  - RT-Transformer
  - RT-Transformer web server
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# retention-time-prediction-fine-tuning

## Summary

Transfer a pre-trained RT-Transformer (graph attention network + 1D-Transformer trained on the 80,038-molecule SMRT dataset) to a new chromatographic method by freezing the encoders and fine-tuning a re-initialized output block on a small PredRet-style retention time dataset.

## When to use

Use when predicting liquid chromatography retention times for a new chromatographic method from molecular structures (InChI/SMILES), especially with limited labeled data where training a retention-time model from scratch is infeasible.

## When NOT to use

- When no pre-trained checkpoint applies and you have a large labeled dataset for the target method — retrain from scratch with train.py instead
- When the input lacks structural identifiers (InChI/SMILES) convertible to molecular graphs and fingerprints — the model requires graph and fingerprint inputs, not tabular metadata
- When the target task is not liquid chromatography retention time prediction (e.g. mass spectra prediction or general property regression)

## Inputs

- CSV dataset with 'InChI' and 'RT' columns (molecular structures and measured retention times), named data.csv at the repository root
- Pre-trained RT-Transformer checkpoint (best_state_dict.pth or best_state_download_dict.pth) downloaded from the Hugging Face space
- SMRT dataset (80,038 molecules) for pre-training/retraining, or PredRet datasets for transfer benchmarks

## Outputs

- Fine-tuned RT-Transformer model predicting retention times under the target chromatographic method
- Per-fold R2 scores and average absolute error (e.g. aggregated r2_distribution.csv across dataset/seed/fold combinations)
- Retention time predictions for new metabolites via test.py, QuickStart.ipynb, or the Hugging Face web server

## How to apply

Load a pre-trained RT-Transformer checkpoint from the 01dadada/RT-Transformer repository (best_state_dict.pth trained on full SMRT, or best_state_download_dict.pth trained on retained molecules only) and prepare your dataset as a CSV with 'InChI' and 'RT' columns. Freeze the pre-trained ResGAT and 1D-Transformer encoder weights and re-initialize the output block, then fine-tune only that block on the target dataset with 10-fold cross-validation, repeated with multiple random seeds (e.g. five seeds, 50 runs per dataset in the PredRet study). Evaluate with R2 per held-out fold and average absolute error; in the article, the full-SMRT pre-trained model was more stable across 41 PredRet datasets (no R2 below -0.5) and outperformed the retained-molecules model when R2 exceeded 0.75. Pre-training rationale: different chromatographic conditions yield different retention times for the same metabolite, so transfer learning from the large SMRT dataset provides scalability across methods.

## Related tools

- **RT-Transformer** (Deep neural network (graph attention network + 1D-Transformer) used both for pre-training on SMRT and for transfer-learning fine-tuning to new chromatographic methods) — https://github.com/01dadada/RT-Transformer
- **RT-Transformer web server** (Hosts the pre-trained model files for download and provides a browser interface for predicting retention times on user data) — https://huggingface.co/spaces/Xue-Jun/RT-Transformer
- **PredRet** (Source of the 41 external retention time datasets used as transfer-learning targets in the benchmark)

## Examples

```
python transfer.py  # after placing data.csv (InChI, RT columns) at repo root and downloading best_state_dict.pth from the Hugging Face space
```

## Evaluation signals

- Per-dataset R2 distributions show no value below -0.5 when using the full-SMRT pre-trained checkpoint (stability criterion from the 41-PredRet benchmark)
- Average absolute error is competitive with published baselines (SMRT benchmark: 27.30 after removing not-retained molecules, 33.41 without removal)
- Fine-tuned model outperforms the retained-molecules pre-trained variant when R2 exceeds 0.75
- Output block converges across repeated seeds (e.g. 5 seeds x 10 folds), indicating transfer rather than overfitting to a single split
- Predicted RTs correlate with measured RTs on held-out folds of the target dataset's cross-validation

## Limitations

- Chromatographic conditions differ between datasets, so the same metabolite can have different retention times; transferability across methods is the core challenge and performance varies by dataset (some PredRet transfers yield low R2)
- Only the output block is fine-tuned in the described protocol; encoder weights are frozen, which may limit adaptation to very different stationary phases
- The full-SMRT pre-trained model is more stable, but the retained-molecules checkpoint can outperform it when R2 > 0.75 — checkpoint choice is a trade-off
- No changelog is available for the 01dadada/RT-Transformer repository, so version drift of checkpoints and code cannot be tracked

## Evidence

- [abstract] we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] The pre-trained RT-Transformer was further transferred to 5 datasets corresponding to different chromatographic conditions and fine-tuned.: "The pre-trained RT-Transformer was further transferred to 5 datasets corresponding to different chromatographic conditions and fine-tuned."
- [readme] Prepare your dataset as a csv file which has "InChI" and "RT" columns.: [evidence span withheld — anchor status: external_doc]
- [readme] best_state_dict.pth The Best model of RT-Transformer train from full data.: [evidence span withheld — anchor status: external_doc]
- [readme] Datasets for transfer learning is download from [PredRet](http://predret.org/): [evidence span withheld — anchor status: external_doc]
