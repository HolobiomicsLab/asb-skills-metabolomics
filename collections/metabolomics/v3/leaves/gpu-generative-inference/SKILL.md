---
name: gpu-generative-inference
description: Use when a trained DiffSpectra/DMT checkpoint exists and you need to
  generate candidate 3D molecular structures from QM9S test spectra (ir/raman/uv/allspectra)
  and score them against ground-truth structures.
when_to_use_negative:
- You are training a model from scratch or pre-training SpecFormer rather than generating/evaluating
  with an existing checkpoint — use the README's --mode train commands instead.
- The spectra are experimental NMR or mass-spectrometry data — DiffSpectra was developed
  and validated only on simulated IR, Raman, and UV-Vis spectra from QM9S.
- You need unconditional molecular generation (e.g. CDGS/JODO baselines) rather than
  spectra-conditioned structure elucidation.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: DiffSpectra
  role: Generative framework whose eval-mode main.py entrypoint performs the GPU sampling
    and metric computation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network that generates the molecules during reverse-time
    sampling
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder providing patch-embedding conditioning (concatenated for
    allspectra mode) that guides DMT during generation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: PyTorch
  role: Deep-learning backend used to load the checkpoint and run the multi-GPU (CUDA_VISIBLE_DEVICES=0,1)
    denoising loop
- name: RDKit
  role: Used by base_metrics.py to canonically compare generated structures against
    ground truth for exact-match accuracy
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:d550eba25bead13113b452c387d02145d3a80f9146dbad751836e52bef06737b
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Diffusion Molecule Transformer (DMT)
  - PyTorch
  - RDKit
  - SpecFormer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gpu-generative-inference

## Summary

Run spectra-conditioned generative sampling from a trained DiffSpectra (DMT) checkpoint on multi-GPU hardware, reverse-time denoising molecules from t=1 to t=0 conditioned on IR, Raman, UV-Vis, or all spectra, then evaluate exact-match accuracy against ground truth.

## When to use

Use when a trained DiffSpectra/DMT checkpoint exists and you need to generate candidate 3D molecular structures from QM9S test spectra (ir/raman/uv/allspectra) and score them against ground-truth structures.

## When NOT to use

- You are training a model from scratch or pre-training SpecFormer rather than generating/evaluating with an existing checkpoint — use the README's --mode train commands instead.
- The spectra are experimental NMR or mass-spectrometry data — DiffSpectra was developed and validated only on simulated IR, Raman, and UV-Vis spectra from QM9S.
- You need unconditional molecular generation (e.g. CDGS/JODO baselines) rather than spectra-conditioned structure elucidation.

## Inputs

- Trained DiffSpectra DMT checkpoint (e.g. checkpoint 40) from huggingface.co/AzureLeon1/DiffSpectra placed in exp/
- QM9S test spectra (IR, Raman, UV-Vis) from the HuggingFace dataset repo, placed at the path configured as data.root
- Configuration files: configs/diffspectra_qm9s.py and configs/base_qm9.py

## Outputs

- Generated molecules with 2D topology and 3D coordinates, saved via --config.eval.save_mols true
- Top-1 / top-10 exact-match accuracy and similarity metrics written to exp/{exp_name}/eval/
- Optionally, 3D structural evaluation (RMSD) via eval_sampled_mols/rmsd.py

## How to apply

Install the environment via env.sh, place the QM9S dataset from the HuggingFace dataset repo, and point configs/diffspectra_qm9s.py (data.root) at it; place the checkpoint from huggingface.co/AzureLeon1/DiffSpectra in exp/. Launch generation with main.py in --mode eval, selecting the checkpoint (--config.eval.ckpts '40'), the number of molecules to sample per spectrum (--config.eval.num_samples 10000), and the spectral conditioning via --config.data.spectra_version (ir, raman, uv, or allspectra where SpecFormer patch embeddings of all three modalities are concatenated). The model runs the reverse-time denoising process from a standard Gaussian sample backward from t=1 to t=0, parameterized by the SE(3)-equivariant DMT backbone guided by SpecFormer spectral embeddings. Accuracy (top-1/top-10 exact match) and similarity metrics are computed automatically during generation, or manually afterwards with python eval_sampled_mols/base_metrics.py; outputs land in exp/{workdir}/eval/.

## Related tools

- **DiffSpectra** (Generative framework whose eval-mode main.py entrypoint performs the GPU sampling and metric computation) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network that generates the molecules during reverse-time sampling) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder providing patch-embedding conditioning (concatenated for allspectra mode) that guides DMT during generation) — https://github.com/AzureLeon1/DiffSpectra
- **PyTorch** (Deep-learning backend used to load the checkpoint and run the multi-GPU (CUDA_VISIBLE_DEVICES=0,1) denoising loop)
- **RDKit** (Used by base_metrics.py to canonically compare generated structures against ground truth for exact-match accuracy)

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- With allspectra conditioning and the pretrained-SpecFormer workdir, sampling 10,000 molecules at checkpoint 40 should reproduce the reported 40.76% top-1 and 99.49% top-10 accuracy.
- The multi-modal 'allspectra' setting must exceed the best single-spectrum setting (Raman, ~30.3% top-1) by roughly +0.3% in top-1 accuracy.
- Generated molecules and metrics must appear under exp/{exp_name}/eval/ with --config.eval.save_mols true.
- Per-modality ablation ordering should hold: Raman > IR > UV-Vis for single-spectrum top-1 accuracy (UV-Vis near 0.3%).
- Manual re-scoring with python eval_sampled_mols/base_metrics.py should match the metrics computed automatically during generation.

## Limitations

- Validation is primarily computational; no experimental/laboratory validation of generated structures has been performed.
- Results are specific to simulated spectra (QM9S, built on QM9); applicability to NMR or mass spectrometry is unexplored.
- No software versions or hardware requirements beyond the multi-GPU CUDA command are stated; environment must be built from env.sh.

## Evidence

- [methods] The reverse-time denoising process starts from a standard Gaussian sample ... and proceeds backward from t=1 to t=0: [evidence span withheld — anchor status: not_found]
