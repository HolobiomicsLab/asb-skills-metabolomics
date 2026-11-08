---
name: diffusion-sampler-evaluation
description: Use when you have a trained spectra-conditioned DiffSpectra (DMT) checkpoint
  and QM9S test spectra (spectra_version ir/raman/uv/allspectra) and need to generate
  molecules and reproduce accuracy/MCES ablation tables.
when_to_use_negative:
- 'Input spectra are NMR or mass spectrometry: the framework currently focuses only
  on three spectral types (UV-Vis, IR, Raman) and applicability to NMR/MS remains
  unexplored.'
- 'You only need to rescore already-generated molecules: run eval_sampled_mols/base_metrics.py
  directly instead of rerunning the full diffusion sampling loop.'
- 'You need experimentally (laboratory) validated structure elucidation: validation
  to date has been primarily computational on simulated QM9S spectra.'
tools:
- name: DiffSpectra
  role: The spectra-conditioned diffusion framework whose eval mode generates molecules
    from IR/Raman/UV-Vis spectra for scoring
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising backbone that parameterizes the reverse-time process
    from t=1 to t=0 during eval sampling
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder whose patch embeddings (concatenated across modalities for
    'allspectra') condition the sampled generations; pretrained checkpoint required
    for the headline setting
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: RDKit
  role: Used for canonical structure comparison and exact-match/similarity metrics
    via the repository's base_metrics.py
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:fb3ac5ddf05646e7e9dd1e0e5b764aa8bf2f779be1b6cfe5fbb0530f57195e5d
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Diffusion Molecule Transformer (DMT)
  - RDKit
  - SpecFormer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# diffusion-sampler-evaluation

## Summary

Samples molecules from a spectra-conditioned diffusion model (DiffSpectra/DMT) in eval mode and scores them against ground-truth structures using exact-match top-1/top-10 accuracy, MCES, and RMSD. It operationalizes multi-modality ablation questions (e.g., IR vs Raman vs UV-Vis vs allspectra conditioning) on the QM9S benchmark.

## When to use

Use when you have a trained spectra-conditioned DiffSpectra (DMT) checkpoint and QM9S test spectra (spectra_version ir/raman/uv/allspectra) and need to generate molecules and reproduce accuracy/MCES ablation tables.

## When NOT to use

- Input spectra are NMR or mass spectrometry: the framework currently focuses only on three spectral types (UV-Vis, IR, Raman) and applicability to NMR/MS remains unexplored.
- You only need to rescore already-generated molecules: run eval_sampled_mols/base_metrics.py directly instead of rerunning the full diffusion sampling loop.
- You need experimentally (laboratory) validated structure elucidation: validation to date has been primarily computational on simulated QM9S spectra.

## Inputs

- QM9S test spectra (IR, Raman, UV-Vis) from huggingface.co/datasets/AzureLeon1/DiffSpectra, placed in the configured data.root directory
- Trained DiffSpectra (DMT) checkpoint from huggingface.co/AzureLeon1/DiffSpectra, placed in the exp/ directory
- Pretrained SpecFormer checkpoint (exp/pretrained_specformer.ckpt) for the allspectra_pretrained_specformer setting
- Configuration files configs/diffspectra_qm9s.py and configs/base_qm9.py

## Outputs

- 10,000 generated 3D molecules per spectrum setting, saved via --config.eval.save_mols true
- Top-1 and top-10 exact-match accuracy and similarity metrics per spectra_version
- MCES distance summary per setting (allspectra target: lowest MCES ~1.780)
- RMSD against ground-truth 3D structures (Hungarian-algorithm atom mapping)
- Metrics and molecules written to exp/{exp_name}/eval/

## How to apply

Run generation in eval mode by invoking main.py with --mode eval, setting --config.data.spectra_version to one of 'ir', 'raman', 'uv', or 'allspectra', --config.eval.num_samples to 10000 molecules per spectrum, and --config.eval.ckpts '40' to select the checkpoint; sampling proceeds via the reverse-time denoising process from t=1 to t=0 starting from a standard Gaussian, with the spectrum-conditioned DMT backbone (use the pretrained SpecFormer checkpoint for the allspectra_pretrained_specformer workdir). Repeat the eval for each spectra_version to build the ablation: among single modalities expect Raman best (~30.3% top-1), then IR (~25.9%), then UV-Vis (~0.3%), with the combined 'allspectra' conditioning targeting the headline 40.76% top-1 / 99.49% top-10. Exact-match top-1/top-10 accuracy and similarity metrics are computed automatically during generation, or manually via eval_sampled_mols/base_metrics.py; 3D quality is assessed with eval_sampled_mols/rmsd.py, which uses Hungarian-algorithm atom mapping against ground-truth conformations. Decision point: if allspectra top-1 does not exceed the best single-spectrum setting (Raman) by roughly +0.3% and match the headline accuracy within tolerance, suspect checkpoint selection, dataset path misconfiguration, or missing SpecFormer pretraining before trusting the ablation.

## Related tools

- **DiffSpectra** (The spectra-conditioned diffusion framework whose eval mode generates molecules from IR/Raman/UV-Vis spectra for scoring) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising backbone that parameterizes the reverse-time process from t=1 to t=0 during eval sampling) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder whose patch embeddings (concatenated across modalities for 'allspectra') condition the sampled generations; pretrained checkpoint required for the headline setting) — https://github.com/AzureLeon1/DiffSpectra
- **RDKit** (Used for canonical structure comparison and exact-match/similarity metrics via the repository's base_metrics.py)

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- The full 'allspectra' setting reproduces the headline 40.76% top-1 and 99.49% top-10 accuracy on QM9S.
- Multi-modal 'allspectra' top-1 accuracy exceeds the best single-spectrum setting (Raman, ~30.3%) by approximately +0.3% and achieves the lowest MCES (~1.780).
- Single-modality results follow the expected ordering: Raman (~30.3%) > IR (~25.9%) >> UV-Vis (~0.3% top-1).
- Accuracy/similarity metrics are auto-computed alongside generation and written to exp/{exp_name}/eval/; manually recomputed values from eval_sampled_mols/base_metrics.py agree.
- RMSD from eval_sampled_mols/rmsd.py is finite and computed via Hungarian-algorithm atom mapping against ground-truth 3D structures.

## Limitations

- Evaluation is restricted to the QM9S dataset (QM9 augmented with simulated spectra), so results do not transfer to experimental spectra or larger molecules.
- UV-Vis alone is nearly uninformative for structure elucidation (~0.3% top-1), so single-UV-Vis ablations are weak baselines.
- Sampling 10,000 molecules per spectrum per setting is computationally expensive and multi-GPU (CUDA_VISIBLE_DEVICES=0,1).
- The framework's validation is primarily computational; no experimental/laboratory validation has been performed.

## Evidence

- [methods] The reverse-time denoising process starts from a standard Gaussian sample ... and proceeds backward from t=1 to t=0: [evidence span withheld — anchor status: not_found]
