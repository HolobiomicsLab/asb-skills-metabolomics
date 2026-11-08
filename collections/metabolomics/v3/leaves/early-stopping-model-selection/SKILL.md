---
name: early-stopping-model-selection
description: Use when fine-tuning a generative chemical language model on a train
  fold (e.g., 90% of known metabolites) and you need an automatic, loss-based stopping
  criterion rather than a fixed number of epochs.
when_to_use_negative:
- The model will be used purely generatively after pretraining and no labeled/fine-tuning
  split exists — there is no validation loss to monitor, so early stopping is inapplicable
- Input is a fixed-epoch benchmark reproduction where the paper specifies an exact
  epoch count and no validation-based stopping criterion
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: PyTorch
  role: Trains the LSTM autoregressively over SMILES tokens and computes validation
    loss for the early-stopping check; GPU access speeds the end-end workflow
  repo: https://github.com/skinniderlab/CLM
- name: Snakemake
  role: Orchestrates the train_models_RNN rule that runs per-seed training and writes
    checkpoints and loss files
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: Runtime for the CLM training and evaluation code
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:1b0a0b9240c9d13b4187ea785fa696031d350a97298ffbefc7b1ceae59cda969
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - PyTorch
  - Python
  - Python 3.10
  - Snakemake
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# early-stopping-model-selection

## Summary

Selects when to stop fine-tuning a chemical language model (e.g., DeepMet's 3-layer LSTM) by monitoring validation loss and halting training before overfitting, then exporting the checkpoint and loss curve. In the DeepMet workflow this is applied per HMDB cross-validation fold and per training seed.

## When to use

Use when fine-tuning a generative chemical language model on a train fold (e.g., 90% of known metabolites) and you need an automatic, loss-based stopping criterion rather than a fixed number of epochs.

## When NOT to use

- The model will be used purely generatively after pretraining and no labeled/fine-tuning split exists — there is no validation loss to monitor, so early stopping is inapplicable
- Input is a fixed-epoch benchmark reproduction where the paper specifies an exact epoch count and no validation-based stopping criterion

## Inputs

- config.yaml with model_params (layers, hidden size 1024, embedding 128, epochs, batch size) and train_seeds entries
- HMDB cross-validation training folds of known metabolite SMILES, each split 0.9 train/validation
- Pre-trained LSTM model checkpoint (ChEMBL pretraining)

## Outputs

- Fine-tuned model checkpoints, one per cross-validation fold and per train_seeds entry
- Loss files recording training and validation loss curves used for the early-stopping decision

## How to apply

Read model hyperparameters (number of layers, epochs, batch size) from the model_params section of config.yaml, then pre-train the 3-layer LSTM (hidden size 1024, embedding 128) autoregressively over SMILES tokens in PyTorch. For each HMDB cross-validation training fold, internally split the fold 0.9 train/validation so that a held-out validation set exists to monitor loss. Train one independent model per train_seeds entry in config.yaml, and at each epoch evaluate validation loss, stopping fine-tuning when the early-stopping criterion triggers (validation loss stops improving) to prevent overfitting to the small metabolite training set. Write out the trained model checkpoints and loss files via the train_models_RNN rule of the Snakemake workflow. Because the cross-validation scheme withholds ~10% of known metabolites at a time to simulate discovery of unknown metabolites, stopping on validation loss preserves generalization to unseen structures rather than memorization.

## Related tools

- **PyTorch** (Trains the LSTM autoregressively over SMILES tokens and computes validation loss for the early-stopping check; GPU access speeds the end-end workflow) — https://github.com/skinniderlab/CLM
- **Snakemake** (Orchestrates the train_models_RNN rule that runs per-seed training and writes checkpoints and loss files) — https://github.com/skinniderlab/CLM
- **Python 3.10** (Runtime for the CLM training and evaluation code) — https://github.com/skinniderlab/CLM

## Examples

```
snakemake --use-conda --cores 1 output/fold_0/model.pt  # runs preprocess -> create_training_sets -> train_models_RNN with early stopping on validation loss
```

## Evaluation signals

- Validation loss decreases and plateaus/rises after the stopping point — the selected checkpoint should correspond to the validation-loss minimum, not the final epoch
- One independent checkpoint and loss file is produced per train_seeds entry in config.yaml, as required by the train_models_RNN rule
- The 0.9 train/validation split within each HMDB fold is respected — no withheld cross-validation test metabolites leak into the validation split used for stopping
- Trained models generate valid, metabolite-like SMILES for withheld metabolites, consistent with the 10%-withheld cross-validation design simulating unknown metabolite discovery

## Limitations

- Early stopping depends on the internal 0.9 train/validation split of each fold; with small folds, validation loss may be a noisy stopping signal
- The model was trained and evaluated on human metabolite structures from HMDB; transfer to evolutionarily distant metabolomes (e.g., plant or bacterial metabolism) will likely require bespoke models, so stopping criteria tuned on human data may not carry over

## Evidence

- [methods] workflow description from source card: [evidence span withheld — anchor status: not_found]
- [methods] rationale for early stopping: [evidence span withheld — anchor status: not_found]
- [readme] framework: [evidence span withheld — anchor status: external_doc]
