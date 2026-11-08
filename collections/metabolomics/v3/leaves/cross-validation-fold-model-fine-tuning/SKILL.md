---
name: cross-validation-fold-model-fine-tuning
description: 'Use when reproducing the DeepMet training protocol: you have a pre-trained
  SMILES language model, a known-metabolite structure set (HMDB), and need per-fold
  fine-tuned generators whose withheld 10% mimics unknown metabolites.'
when_to_use_negative:
- If you only want to annotate metabolomics data with an already-trained model — the
  article provides the trained DeepMet model for that purpose, so no fine-tuning is
  needed
- If the target organism is evolutionarily distant from humans (e.g. plant or bacterial
  metabolism) — the article notes such applications 'will likely require' bespoke
  models rather than the human-metabolite-trained folds
- If the task is candidate prioritization or MS/MS annotation rather than model training
  — use the mass-to-charge filtering and meta-learning skills instead
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Autoregressive pre-training and per-fold fine-tuning of the 3-layer LSTM (hidden
    1024, embedding 128) over SMILES tokens; GPU access makes the end-end workflow
    take ~5 minutes
- name: Snakemake
  role: Orchestrates the workflow; the train_models_RNN rule trains one RNN model
    per train_seeds entry on each input fold and writes checkpoints and loss files
- name: CLM
  role: Repository containing the Snakefile, config.yaml (model_params and train_seeds),
    and Python source code required to train and evaluate the chemical language models
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:ccd081e262a7ca8c41294c30b94554aa487ec437a4f2ed868742e05652b74aeb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - CLM
  - DeepMet
  - PyTorch
  - Python
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-validation-fold-model-fine-tuning

## Summary

Fine-tunes a pre-trained chemical language model (a 3-layer LSTM, hidden size 1024, embedding 128) independently on each cross-validation fold of known metabolite structures (e.g. HMDB), holding out 10% of metabolites per fold to simulate discovery of unknown metabolites.

## When to use

Use when reproducing the DeepMet training protocol: you have a pre-trained SMILES language model, a known-metabolite structure set (HMDB), and need per-fold fine-tuned generators whose withheld 10% mimics unknown metabolites.

## When NOT to use

- If you only want to annotate metabolomics data with an already-trained model — the article provides the trained DeepMet model for that purpose, so no fine-tuning is needed
- If the target organism is evolutionarily distant from humans (e.g. plant or bacterial metabolism) — the article notes such applications 'will likely require' bespoke models rather than the human-metabolite-trained folds
- If the task is candidate prioritization or MS/MS annotation rather than model training — use the mass-to-charge filtering and meta-learning skills instead

## Inputs

- Pre-trained DeepMet LSTM checkpoint (3 layers, hidden 1024, embedding 128) trained autoregressively on SMILES tokens
- Known metabolite SMILES set (HMDB), preprocessed to canonical SMILES and split into cross-validation train/test folds with .vocabulary files
- config.yaml with model_params (layers, epochs, batch size) and train_seeds entries

## Outputs

- One fine-tuned model checkpoint per cross-validation fold and per train_seeds entry
- Per-model loss files recording training and validation loss (used for early stopping)
- Models capable of generating metabolite-like SMILES for withheld-fold evaluation and downstream candidate generation

## How to apply

First read the network architecture and training settings (layers, epochs, batch size) from the model_params section of config.yaml in the skinniderlab/CLM repository. Pre-train the 3-layer LSTM autoregressively over SMILES tokens with PyTorch (GPU access recommended — the full workflow takes ~5 minutes on GPU machines). Create cross-validation folds with the create_training_sets Snakemake rule, which splits input SMILES into train/test fold files and generates .vocabulary files; DeepMet withheld 10% of known metabolites per fold to simulate discovery of unknown metabolites. For each fold, internally split the training data into a 0.9 train/validation split, fine-tune the pre-trained model on the train portion, and apply early stopping on validation loss to decide when to stop. Train one independent model per train_seeds entry in config.yaml via the train_models_RNN rule, writing out model checkpoints and loss files for each seed/fold combination.

## Related tools

- **PyTorch** (Autoregressive pre-training and per-fold fine-tuning of the 3-layer LSTM (hidden 1024, embedding 128) over SMILES tokens; GPU access makes the end-end workflow take ~5 minutes)
- **Snakemake** (Orchestrates the workflow; the train_models_RNN rule trains one RNN model per train_seeds entry on each input fold and writes checkpoints and loss files)
- **CLM** (Repository containing the Snakefile, config.yaml (model_params and train_seeds), and Python source code required to train and evaluate the chemical language models) — https://github.com/skinniderlab/CLM

## Examples

```
git clone https://github.com/skinniderlab/CLM.git && cd CLM && snakemake train_models_RNN
```

## Evaluation signals

- Each fold yields an independent checkpoint per train_seeds entry in config.yaml (number of models = folds × seeds)
- Validation-loss curves exist per model and early stopping terminated fine-tuning based on the 0.9 train/validation split within each fold
- Held-out 10% of metabolites per fold were never seen during fine-tuning, so generation can be scored for recapitulation of withheld structures
- Downstream check: the model recapitulates well-studied metabolites that were absent from or misannotated within HMDB, as the article reports
- .vocabulary files generated by create_training_sets match the token space used by the fine-tuned checkpoints

## Limitations

- The model was trained and evaluated on human metabolite structures; evolutionarily distant applications (e.g. plant or bacterial metabolism) will likely require bespoke models
- The approach has inherent limitations in anticipating metabolites originating from divergent, as-of-yet undiscovered biosynthetic routes
- Fine-tuning and evaluation are only meaningful in the cross-validation scheme; using a single split without withheld folds loses the 'simulate discovery of unknown metabolites' rationale

## Evidence
