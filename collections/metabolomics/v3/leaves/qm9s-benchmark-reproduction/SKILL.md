---
name: qm9s-benchmark-reproduction
description: Use when validating a spectral-structure generative model on QM9S simulated
  IR/Raman/UV-Vis spectra, or when comparing a new elucidation model against DiffSpectra's
  40.76% top-1 / 99.49% top-10 baseline.
when_to_use_negative:
- Input spectra are experimental NMR or mass spectrometry data — the paper explicitly
  limits the framework to IR, Raman, and UV-Vis modalities, so extension to NMR/MS
  remains unexplored.
- The goal is unconditional de novo molecule generation rather than spectra-conditioned
  elucidation — the eval pipeline conditions every sample on a QM9S spectrum and scores
  against a ground-truth reference structure.
- Only a single spectrum type is available and you still expect the 40.76% headline
  number — that figure corresponds to the all-spectra (IR + Raman + UV-Vis) configuration
  with pretrained SpecFormer; single-modality runs (ir, raman, uv) yield lower accuracy.
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: The generative framework being benchmarked; its `main.py --mode eval` entry
    point runs the reverse-time diffusion sampling from QM9S spectra and computes
    accuracy/similarity metrics automatically.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network that generates the 2D topology and 3D
    coordinates of candidate molecules during sampling.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder that embeds the QM9S IR/Raman/UV-Vis spectra to condition
    the diffusion process; the pretrained checkpoint `pretrained_specformer.ckpt`
    is required for the headline configuration.
  repo: https://huggingface.co/AzureLeon1/DiffSpectra
- name: eval_sampled_mols/rmsd.py
  role: Post-generation script that computes 3D structural RMSD by atom-mapping generated
    molecules to ground-truth 3D structures via the Hungarian algorithm.
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:05e1015b267c44d1ae648948ca9a58049512b857fb53089505bac3a3f707d103
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
  - eval_sampled_mols/rmsd.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# qm9s-benchmark-reproduction

## Summary

Reproduce the DiffSpectra benchmark on the QM9S test set: run the spectra-conditioned diffusion sampler (DMT + SpecFormer) for a fixed number of samples per spectrum, then compute top-1/top-10 accuracy, similarity, and 3D RMSD metrics exactly as defined in the paper.

## When to use

Use when validating a spectral-structure generative model on QM9S simulated IR/Raman/UV-Vis spectra, or when comparing a new elucidation model against DiffSpectra's 40.76% top-1 / 99.49% top-10 baseline.

## When NOT to use

- Input spectra are experimental NMR or mass spectrometry data — the paper explicitly limits the framework to IR, Raman, and UV-Vis modalities, so extension to NMR/MS remains unexplored.
- The goal is unconditional de novo molecule generation rather than spectra-conditioned elucidation — the eval pipeline conditions every sample on a QM9S spectrum and scores against a ground-truth reference structure.
- Only a single spectrum type is available and you still expect the 40.76% headline number — that figure corresponds to the all-spectra (IR + Raman + UV-Vis) configuration with pretrained SpecFormer; single-modality runs (ir, raman, uv) yield lower accuracy.

## Inputs

- Preprocessed QM9S dataset (simulated IR, Raman, UV-Vis spectra + ground-truth 2D/3D molecules) from huggingface.co/datasets/AzureLeon1/DiffSpectra
- Trained DiffSpectra / DMT checkpoint (e.g. exp/pretrained_specformer.ckpt or released checkpoint 40) from huggingface.co/AzureLeon1/DiffSpectra
- Config files configs/diffspectra_qm9s.py and configs/base_qm9.py

## Outputs

- Generated molecules and evaluation metrics under exp/{exp_name}/eval/
- Accuracy and similarity scores (top-1, top-10) computed automatically during generation
- 3D structural RMSD table from eval_sampled_mols/rmsd.py (Hungarian-mapped atom correspondence)

## How to apply

Load the QM9S test split (preprocessed IR, Raman, UV-Vis spectra paired with ground-truth 3D molecules) from the HuggingFace dataset, set `data.root` in `configs/diffspectra_qm9s.py`, and either train from scratch (e.g. `spectra_version allspectra`, optionally pointing `pretrained_specformer_path` at `exp/pretrained_specformer.ckpt`) or download the released checkpoint into `exp/`. Run the evaluation mode with `--config.eval.ckpts '40'`, `--config.eval.num_samples 10000`, and `--config.eval.save_mols true` so the reverse-time diffusion process generates molecules from a standard Gaussian sample guided by SpecFormer spectral embeddings; accuracy and similarity metrics are computed automatically during generation, and 3D structural quality is obtained afterwards via `eval_sampled_mols/rmsd.py`, which atom-maps predictions to the ground-truth 3D structures using the Hungarian algorithm and reports RMSD. Validate against the reported anchors: top-1 accuracy 40.76% and top-10 accuracy 99.49% for the all-spectra pretrained-SpecFormer configuration; if your run diverges materially, first check that the checkpoint index, sample count, and spectra version match the paper's setting before re-tuning.

## Related tools

- **DiffSpectra** (The generative framework being benchmarked; its `main.py --mode eval` entry point runs the reverse-time diffusion sampling from QM9S spectra and computes accuracy/similarity metrics automatically.) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network that generates the 2D topology and 3D coordinates of candidate molecules during sampling.) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder that embeds the QM9S IR/Raman/UV-Vis spectra to condition the diffusion process; the pretrained checkpoint `pretrained_specformer.ckpt` is required for the headline configuration.) — https://huggingface.co/AzureLeon1/DiffSpectra
- **eval_sampled_mols/rmsd.py** (Post-generation script that computes 3D structural RMSD by atom-mapping generated molecules to ground-truth 3D structures via the Hungarian algorithm.) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Top-1 accuracy ≈ 40.76% and top-10 accuracy ≈ 99.49% for the all-spectra + pretrained SpecFormer configuration.
- `--config.eval.num_samples 10000` and `--config.eval.ckpts '40'` are set exactly as in the README evaluation commands.
- Generated molecules and metrics land in `exp/{exp_name}/eval/` and accuracy/similarity scores are produced automatically alongside generation.
- 3D RMSD is computed only after Hungarian atom-mapping against the corresponding ground-truth 3D structures, not by naive coordinate comparison.
- Running `python eval_sampled_mols/base_metrics.py` manually should reproduce the automatically computed accuracy/similarity numbers.

## Limitations

- Validation is primarily computational; no experimental or laboratory validation of elucidated structures has been performed.
- The framework covers only three spectral types (UV-Vis, IR, Raman); applicability to NMR or MS remains to be investigated.
- Results depend on the QM9S dataset, which augments QM9 with simulated spectra — real experimental spectra may shift the achievable accuracy.
- Model checkpoints must be manually downloaded and placed in `exp/`; misplacing them or using the wrong checkpoint index (e.g. not '40') silently degrades accuracy.
- Evaluation uses two GPUs (`CUDA_VISIBLE_DEVICES=0,1`); a single-GPU run may be slower or require batch-size adjustments not covered in the README.

## Evidence

- [methods] The reverse-time sampling procedure that the eval mode executes.: [evidence span withheld — anchor status: not_found]
