---
name: rnn-language-model-training-on-smiles
description: Use when a set of preprocessed, canonicalized SMILES is ready and you
  need to train one or more RNN generators per train/test fold (e.g. DeepMet-style
  pre-training then HMDB fine-tuning with early stopping).
when_to_use_negative:
- You only want to sample/annotate with an already-trained DeepMet model — the article
  notes this does not require re-training
- Your data is non-SMILES molecular representations or requires a different architecture
  (e.g. the AddCarbon model, which is trained in a separate workflow step)
- You have no train/test fold structure or config.yaml with model_params and train_seeds
  defined
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3336
tools:
- name: DeepMet
  role: 'The chemical language model produced by this training procedure: a 3-layer
    LSTM trained autoregressively on metabolite SMILES'
  repo: https://github.com/skinniderlab/CLM
- name: PyTorch
  role: Deep learning framework (`torch`) used to implement and train the LSTM; GPU
    access makes the end-end workflow take around 5 minutes
- name: Snakemake
  role: Workflow engine; the `train_models_RNN` rule orchestrates training one or
    more RNN models on an input fold
- name: CLM
  role: Repository containing the Python source code, Snakefile, and config.yaml required
    to train and evaluate chemical language models
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:a8eb8bb7359fd60785fcda94404c036675db0998bb64d3f9746ee3c0c6c2b4e1
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rnn-language-model-training-on-smiles

## Summary

Train an autoregressive RNN (LSTM) chemical language model on metabolite SMILES via the CLM Snakemake workflow: pre-train on a general corpus, then fine-tune on HMDB training folds with early stopping, producing checkpoints and loss files for later metabolite generation. Grounded in the DeepMet configuration (3-layer LSTM, hidden 1024, embedding 128) trained with PyTorch.

## When to use

Use when a set of preprocessed, canonicalized SMILES is ready and you need to train one or more RNN generators per train/test fold (e.g. DeepMet-style pre-training then HMDB fine-tuning with early stopping).

## When NOT to use

- You only want to sample/annotate with an already-trained DeepMet model — the article notes this does not require re-training
- Your data is non-SMILES molecular representations or requires a different architecture (e.g. the AddCarbon model, which is trained in a separate workflow step)
- You have no train/test fold structure or config.yaml with model_params and train_seeds defined

## Inputs

- File of canonical (optionally augmented) SMILES per training fold from the `create_training_sets` step
- `.vocabulary` files for the training SMILES
- `config.yaml` containing `model_params` (layers, hidden/embedding size, epochs, batch size) and `train_seeds` entries

## Outputs

- One trained RNN model checkpoint per `train_seeds` entry per input fold
- Training/validation loss files for each trained model

## How to apply

Run the `train_models_RNN` rule of the Snakemake `Snakefile`, which trains one independent RNN per `train_seeds` entry defined in `config.yaml`; read the network architecture and training settings (e.g. number of layers, epochs, batch size) from the `model_params` section of that config. Following DeepMet, use a 3-layer LSTM autoregressively over SMILES tokens with hidden size 1024 and embedding size 128, implemented in PyTorch (with GPU access, the end-end workflow takes around 5 minutes). Pre-train the model on a general SMILES corpus, then fine-tune on each HMDB cross-validation training fold using an internal 0.9 train/validation split of that fold, applying the early-stopping criterion on validation loss to decide when to stop. The rule writes out the trained model checkpoints and loss files as its Snakemake outputs, and the fold structure comes from the upstream `create_training_sets` step that splits SMILES into train/test fold files with `.vocabulary` files for the training SMILES.

## Related tools

- **DeepMet** (The chemical language model produced by this training procedure: a 3-layer LSTM trained autoregressively on metabolite SMILES) — https://github.com/skinniderlab/CLM
- **PyTorch** (Deep learning framework (`torch`) used to implement and train the LSTM; GPU access makes the end-end workflow take around 5 minutes)
- **Snakemake** (Workflow engine; the `train_models_RNN` rule orchestrates training one or more RNN models on an input fold)
- **CLM** (Repository containing the Python source code, Snakefile, and config.yaml required to train and evaluate chemical language models) — https://github.com/skinniderlab/CLM

## Examples

```
snakemake --use-conda --cores 1 -- until train_models_RNN
```

## Evaluation signals

- Snakemake output files (model checkpoints and loss files) for the `train_models_RNN` rule exist for every combination of fold and `train_seeds` entry
- Number of trained models equals the number of `train_seeds` entries in `config.yaml`
- Early stopping fired on validation loss — final checkpoints correspond to the minimum validation loss, not simply the last epoch
- Downstream sampling from each checkpoint yields valid SMILES consistent with the `.vocabulary` files from `create_training_sets`
- Loss files show pre-training and per-fold fine-tuning curves with no divergence between the 0.9 train/validation split

## Limitations

- Cross-validation folds withhold 10% of known metabolites at a time, so each model sees only 90% of the known metabolome during fine-tuning
- The end-to-end workflow's ~5 min runtime assumes `torch` has GPU access; CPU-only training will be far slower
- Model parameters (layers, hidden size, epochs, batch size) must be set explicitly in `model_params` of config.yaml — defaults are not documented here
- SMILES should be preprocessed (canonicalization, optional enum-factor augmentation) upstream; training directly on raw SMILES is not what this rule expects

## Evidence

- [other] Write out checkpoints and loss files via train_models_RNN: [evidence span withheld — anchor status: not_found]
- [other] LSTM architecture and training settings in config.yaml: [evidence span withheld — anchor status: not_found]
- [readme] CLM repo purpose: [evidence span withheld — anchor status: external_doc]
- [other] Early stopping on validation loss: [evidence span withheld — anchor status: not_found]
- [other] Internal 0.9 train/validation split of each fold: [evidence span withheld — anchor status: not_found]
