---
name: diffusion-sampling-ablation-comparison
description: 'Use when reproducing the DiffSpectra 2D-only vs joint 2D/3D ablation:
  you have the QM9S test spectra and trained checkpoints and need to quantify how
  the 3D coordinate stream changes exact-structure accuracy and similarity metrics.'
when_to_use_negative:
- The goal is a spectra-modality ablation (IR-only, Raman-only, UV-Vis-only) rather
  than the 2D-vs-3D modality comparison — use the README's spectra_version variants
  instead
- Benchmarking against unconditional molecular generation models (CDGS, JODO) — those
  require a different evaluation setup without spectral conditioning
- Training new models from scratch — this skill covers sampling and evaluation of
  existing checkpoints, not the training or SpecFormer pre-training pipeline
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0092
tools:
- name: DiffSpectra
  role: Conditional diffusion framework whose sampler is run twice (joint and X-ablated)
    in 'allspectra' mode to produce the two variants being compared
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network (data prediction model d_theta) parameterizing
    the reverse process over H, A, and X streams; disabling the X stream yields the
    2D-only ablation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder providing IR/Raman/UV-Vis embeddings that condition both
    ablation arms identically
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:9346a08342af24f4937790b5b00d4584bac7e2bdc65d4546b1e4946c3900a7cb
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# diffusion-sampling-ablation-comparison

## Summary

Ablation protocol for DiffSpectra that contrasts joint 2D/3D diffusion over node features H, edge features A, and 3D coordinates X against a 2D-only topological variant, to quantify how 3D geometric modeling improves exact-structure elucidation from spectra.

## When to use

Use when reproducing the DiffSpectra 2D-only vs joint 2D/3D ablation: you have the QM9S test spectra and trained checkpoints and need to quantify how the 3D coordinate stream changes exact-structure accuracy and similarity metrics.

## When NOT to use

- The goal is a spectra-modality ablation (IR-only, Raman-only, UV-Vis-only) rather than the 2D-vs-3D modality comparison — use the README's spectra_version variants instead
- Benchmarking against unconditional molecular generation models (CDGS, JODO) — those require a different evaluation setup without spectral conditioning
- Training new models from scratch — this skill covers sampling and evaluation of existing checkpoints, not the training or SpecFormer pre-training pipeline

## Inputs

- QM9S test-set spectra (IR, Raman, UV-Vis) from huggingface.co/datasets/AzureLeon1/DiffSpectra
- Trained DiffSpectra checkpoint from huggingface.co/AzureLeon1/DiffSpectra placed in exp/
- Config files configs/diffspectra_qm9s.py and configs/base_qm9.py

## Outputs

- Generated molecule sets (SMILES and structures) for the joint 2D+3D and 2D-only variants under exp/{exp_name}/eval/
- Metrics table (CSV) with rows per variant and columns for top-1 accuracy, top-10 accuracy, TaniSim_MG/TaniSim_MA/FGSim similarities, and RMSD

## How to apply

Run the DiffSpectra sampler in 'allspectra' mode (IR + Raman + UV-Vis conditioning) with the DMT data-prediction model d_theta and self-conditioning, performing reverse-time denoising from a standard Gaussian sample from t=1 to t=0 with config.eval.num_samples=10000 and config.eval.ckpts='40' to generate 10,000 conditioned molecules. Re-run with the coordinate stream X disabled (2D-only diffusion over H and A), keeping all other settings identical, to obtain the ablated variant. Compute top-1/top-10 exact-structure accuracy plus fingerprint-based similarities (TaniSim_MG, TaniSim_MA, FGSim) by comparing generated SMILES to ground truth via eval_sampled_mols/base_metrics.py, mirroring Table 2. For the joint 2D+3D variant only, align generated coordinates to ground truth via Hungarian atom mapping and record RMSD with eval_sampled_mols/rmsd.py. Aggregate both variants into a single CSV (rows = variants; columns = top-1, top-10, similarity, RMSD) and validate against the reported drop from 40.76% to 9.49% top-1.

## Related tools

- **DiffSpectra** (Conditional diffusion framework whose sampler is run twice (joint and X-ablated) in 'allspectra' mode to produce the two variants being compared) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network (data prediction model d_theta) parameterizing the reverse process over H, A, and X streams; disabling the X stream yields the 2D-only ablation) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder providing IR/Raman/UV-Vis embeddings that condition both ablation arms identically) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra; python eval_sampled_mols/base_metrics.py; python eval_sampled_mols/rmsd.py
```

## Evaluation signals

- Joint 2D+3D row reproduces reported accuracies within diffusion sampling tolerance: top-1 ≈ 40.76% and top-10 ≈ 99.49%
- 2D-only row shows the reported degradation: top-1 accuracy falls to ≈ 9.49%
- Similarity metrics improve for the joint variant in the reported directions: TaniSim_MG 0.5609→0.7875, TaniSim_MA 0.8350→0.9262, FGSim 0.9034→0.9652
- RMSD statistics exist only for the joint 2D+3D variant and are computed via Hungarian atom mapping against ground-truth 3D structures
- Output CSV contains one row per variant with all specified columns populated from 10,000 samples (config.eval.num_samples=10000)

## Limitations

- Run-to-run variance in stochastic reverse diffusion sampling means reproduced accuracies will fluctuate; only approximate agreement with reported numbers is expected
- Validation to date is computational — no experimental/laboratory confirmation of elucidated structures
- Limited to QM9-scale molecules with simulated IR, Raman, and UV-Vis spectra; NMR and mass spectrometry modalities are not supported

## Evidence

- [methods] Reverse-time denoising start condition: [evidence span withheld — anchor status: not_found]
