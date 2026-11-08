---
name: se3-equivariant-molecule-denoising-evaluation
description: Use when comparing architecture-based equivariance versus augmentation-based
  equivariance for a 2D/3D diffusion molecule generator, specifically to reproduce
  the DMT Acc@1 contrast (35.64% vs 31.24% vs 20.25%) under checkpoint-40, 10000-sample
  eval settings.
when_to_use_negative:
- The spectral input types are outside IR, Raman, and UV-Vis (e.g., NMR or MS), which
  DiffSpectra's applicability has not yet been demonstrated on.
- You need experimental/laboratory validation — the article states validation to date
  is primarily computational.
- You lack the trained DMT checkpoints or the QM9S dataset; the comparison requires
  rerunning eval at checkpoint 40 with num_samples=10000.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network whose reverse diffusion (t=1 to t=0) generates
    joint 2D topology and 3D coordinates; its accuracy with and without augmentation
    is the quantity compared.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: DiffSpectra
  role: Spectra-conditioned diffusion framework providing the training/eval entry
    point (main.py), configuration files, and evaluation scripts used for both conditions.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder providing spectral embeddings that condition
    DMT during the evaluated denoising runs.
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:11de4e7fc7dd71aa685383311da1d5fc31126082c5dfb12784dfcc0e83ad3ab8
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Diffusion Molecule Transformer (DMT)
  - SpecFormer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 24
---

# se3-equivariant-molecule-denoising-evaluation

## Summary

Evaluates whether explicit SE(3)-equivariant inductive biases in the DMT denoising network (model-based equivariance) outperform learning equivariance via rotation/translation data augmentation (data-based equivariance) for spectra-conditioned molecular structure elucidation on QM9S.

## When to use

Use when comparing architecture-based equivariance versus augmentation-based equivariance for a 2D/3D diffusion molecule generator, specifically to reproduce the DMT Acc@1 contrast (35.64% vs 31.24% vs 20.25%) under checkpoint-40, 10000-sample eval settings.

## When NOT to use

- The spectral input types are outside IR, Raman, and UV-Vis (e.g., NMR or MS), which DiffSpectra's applicability has not yet been demonstrated on.
- You need experimental/laboratory validation — the article states validation to date is primarily computational.
- You lack the trained DMT checkpoints or the QM9S dataset; the comparison requires rerunning eval at checkpoint 40 with num_samples=10000.

## Inputs

- QM9S dataset (QM9 augmented with simulated IR, Raman, UV-Vis spectra) from huggingface.co/datasets/AzureLeon1/DiffSpectra
- Trained DMT model checkpoints (e.g., checkpoint 40) from huggingface.co/AzureLeon1/DiffSpectra
- Configuration files configs/diffspectra_qm9s.py and configs/base_qm9.py with eval settings (ckpts='40', num_samples=10000, spectra_version=allspectra)

## Outputs

- Top-1 accuracy (Acc@1) for the standard configuration with augmentation (reference 35.64%)
- Top-1 accuracy and MCES for the augmentation-removed, equivariance-architecture-only condition (reference 20.25% Acc@1, 3.6036 MCES)
- results_table.csv containing both accuracy conditions plus the 20.25% reference value
- Generated molecules and metrics under exp/{exp_name}/eval/

## How to apply

Load the QM9S dataset from huggingface.co/datasets/AzureLeon1/DiffSpectra and the trained DMT checkpoints from huggingface.co/AzureLeon1/DiffSpectra, configuring spectra-conditioned generation with spectra_version=allspectra as in configs/diffspectra_qm9s.py. Run the reverse denoising process (continuous-time diffusion, t from 1 to 0) with the DMT model in eval mode for 10,000 generated samples (num_samples=10000, checkpoint 40) to get top-1 accuracy for the standard configuration with augmentation. Then remove the augmentation setting from the DMT configuration and rerun the identical evaluation (eval mode, num_samples=10000, checkpoint 40) to isolate architectural equivariance alone. The reported reference values are 35.64% Acc@1 (model-based equivariant DMT, MCES 1.7795), 31.24% for the data-based augmented variant, and 20.25% Acc@1 / 3.6036 MCES when augmentation is removed. Validation hinges on the augmentation-removed run matching 20.25% Acc@1 and the with-augmentation run exceeding it. Write both conditions plus the 20.25% reference into results_table.csv.

## Related tools

- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network whose reverse diffusion (t=1 to t=0) generates joint 2D topology and 3D coordinates; its accuracy with and without augmentation is the quantity compared.) — https://github.com/AzureLeon1/DiffSpectra
- **DiffSpectra** (Spectra-conditioned diffusion framework providing the training/eval entry point (main.py), configuration files, and evaluation scripts used for both conditions.) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder providing spectral embeddings that condition DMT during the evaluated denoising runs.)

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.data.spectra_version allspectra
```

## Evaluation signals

- The augmentation-removed condition's top-1 accuracy should match the reported 20.25% value.
- The with-augmentation model-based condition should exceed the augmentation-removed variant (35.64% > 20.25%), confirming architectural equivariance outperforms data augmentation alone.
- The model-based SE(3)-equivariant implementation should also outperform the data-based augmented approach (35.64% > 31.24%) and show lower MCES (1.7795 vs 3.6036).
- Evaluation parameters must match: mode=eval, ckpts='40', num_samples=10000, spectra_version=allspectra.
- Metrics are computed automatically alongside generation, or manually via eval_sampled_mols/base_metrics.py; 3D structure evaluation via eval_sampled_mols/rmsd.py.

## Limitations

- DiffSpectra currently focuses on three spectral types (UV-Vis, IR, Raman); applicability to NMR or MS remains unexplored.
- Validation is primarily computational; no experimental/laboratory validation has been performed.
- No software versions or implementation details (e.g., diffusion framework or deep learning library) are stated beyond installing the environment via env.sh.
- The reported contrast is specific to QM9S with simulated spectra; generalization to other spectral datasets is not established here.

## Evidence

- [methods] methods: [evidence span withheld — anchor status: not_found]
