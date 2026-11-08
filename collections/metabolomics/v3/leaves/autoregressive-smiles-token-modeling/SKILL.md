---
name: autoregressive-smiles-token-modeling
description: Use when the goal is to generate novel metabolite-like SMILES from a
  corpus of known metabolite structures, or to reproduce/extend DeepMet-style training
  with PyTorch and Snakemake.
when_to_use_negative:
- You only need to annotate metabolomics data with existing predictions — the article
  notes the trained DeepMet model can be used directly instead of retraining.
- Applying the model to evolutionarily distant metabolomes (plant or bacterial metabolism)
  without retraining — the model was trained and evaluated only on human metabolite
  structures.
- The goal is property prediction or embedding (e.g., CDDD continuous descriptors)
  rather than de novo SMILES generation.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implements the autoregressive LSTM training over SMILES tokens (requires GPU
    for reasonable runtime)
- name: Snakemake
  role: Orchestrates the end-to-end workflow, including the train_models_RNN rule
    that writes checkpoints and loss files
- name: DeepMet
  role: The chemical language model being trained; pre-trained on known metabolites
    then fine-tuned per HMDB fold
- name: RDKit
  role: 'Preprocessing: converting to molecules, removing light fragments, neutralizing
    charges, filtering valid elements'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:bb707d37a6faa9755c436ca6be3fa8ededa233264b32897289ba02d7d7a84327
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# autoregressive-smiles-token-modeling

## Summary

Train an autoregressive character-level LSTM chemical language model on SMILES tokens to learn the latent biosynthetic logic of known metabolites, so the model can generate metabolite-like structures. In DeepMet this means pre-training on known metabolite SMILES and fine-tuning per HMDB cross-validation fold.

## When to use

Use when the goal is to generate novel metabolite-like SMILES from a corpus of known metabolite structures, or to reproduce/extend DeepMet-style training with PyTorch and Snakemake.

## When NOT to use

- You only need to annotate metabolomics data with existing predictions — the article notes the trained DeepMet model can be used directly instead of retraining.
- Applying the model to evolutionarily distant metabolomes (plant or bacterial metabolism) without retraining — the model was trained and evaluated only on human metabolite structures.
- The goal is property prediction or embedding (e.g., CDDD continuous descriptors) rather than de novo SMILES generation.

## Inputs

- SMILES strings of known metabolites (e.g., HMDB training folds produced by create_training_sets, with .vocabulary files)
- config.yaml with model_params (layers, hidden size, embedding size, epochs, batch size) and train_seeds entries

## Outputs

- Trained model checkpoints (one per train_seeds entry per cross-validation fold)
- Training/validation loss files from early-stopping monitoring
- A generative model producing metabolite-like SMILES (later used to build the 34.4-million-structure resource)

## How to apply

Read model architecture and training settings (number of layers, epochs, batch size) from the model_params section of config.yaml. Pre-train a 3-layer LSTM (hidden size 1024, embedding size 128) autoregressively over SMILES tokens using PyTorch with GPU access. Fine-tune the pre-trained model on each HMDB cross-validation training fold (10% of known metabolites withheld at a time to simulate discovery of unknown metabolites), internally splitting each fold 0.9 train/validation, and train one independent model per train_seeds entry in config.yaml. Apply early stopping on validation loss to select the stopping point for fine-tuning each model, and write checkpoints and loss files via the train_models_RNN rule of the Snakemake workflow. Before training, preprocess SMILES with RDKit (canonicalization, removing light fragments, neutralizing charges, filtering for valid elements, removing rare SMILES vocabulary tokens), optionally augmenting SMILES by an enum factor.

## Related tools

- **PyTorch** (Implements the autoregressive LSTM training over SMILES tokens (requires GPU for reasonable runtime))
- **Snakemake** (Orchestrates the end-to-end workflow, including the train_models_RNN rule that writes checkpoints and loss files)
- **DeepMet** (The chemical language model being trained; pre-trained on known metabolites then fine-tuned per HMDB fold)
- **RDKit** (Preprocessing: converting to molecules, removing light fragments, neutralizing charges, filtering valid elements)

## Examples

```
snakemake --use-conda --cores 1 train_models_RNN  # after configuring model_params and train_seeds in config.yaml
```

## Evaluation signals

- Validation loss decreases and early stopping fires at a stable epoch across all train_seeds replicates
- One independent trained model (checkpoint + loss file) exists per train_seeds entry per cross-validation fold
- Held-out 10% of known metabolites per fold are not present in that fold's training set (cross-validation integrity check)
- Generated SMILES are chemically valid molecules after RDKit parsing (canonicalization, valid elements, neutralized charges)
- Model recapitulates well-studied metabolites that were absent from or misannotated within HMDB and added in later releases

## Limitations

- Model was trained and evaluated on human metabolite structures; evolutionarily distant applications (plant or bacterial metabolism) likely require bespoke models
- Inherent limitation in anticipating metabolites originating from divergent, as-of-yet undiscovered biosynthetic routes
- Training/evaluation depends on HMDB curation quality; misannotated or absent metabolites in HMDB affect the learned chemical space

## Evidence

- [methods] DeepMet was trained in a cross-validation scheme, with 10% of the known metabolites withheld from the training set at a time to simulate the discovery of unknown metabolites.: [evidence span withheld — anchor status: not_found]
- [methods] train_models_RNN trains RNN models on an input fold: "This step trains one or more (the number of `train_seeds` entries in `config.yaml`) RNN models on an input fold."
- [methods] GPU runtime note: "The end-end workflow should take around 5 minutes on computers where `torch` has access to a gpu"
- [readme] README points to documentation and purpose: [evidence span withheld — anchor status: external_doc]
- [discussion] recapitulation of misannotated HMDB entries: "DeepMet recapitulates the existence of well-studied metabolites that were absent from or misannotated within the HMDB due to curation errors"
