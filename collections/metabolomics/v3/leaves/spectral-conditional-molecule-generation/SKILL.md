---
name: spectral-conditional-molecule-generation
description: Use when you have simulated or measured molecular spectra (IR, Raman,
  UV-Vis, or all jointly) and need de novo molecular structure elucidation evaluated
  by top-1/top-10 exact match rather than retrieval from a fixed library.
when_to_use_negative:
- You need to condition on NMR or mass-spectral data — the article states DiffSpectra
  currently focuses only on UV-Vis, IR, and Raman spectra
- The goal is retrieval of known structures from a fixed library rather than generative,
  de novo elucidation
- You only have a single spectrum type and expect joint-modality accuracy — single-modality
  runs (e.g. UV-Vis at ~0.3% top-1) substantially underperform
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: DiffSpectra
  role: Spectra-conditioned diffusion framework executed in eval mode to generate
    molecules from spectra
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network producing joint 2D/3D molecular structures
    during reverse diffusion
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer spectral encoder whose patch embeddings (concatenated in the allspectra
    setting) condition the denoiser
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: RDKit
  role: Used for molecule validation and canonical-structure comparison when computing
    exact-match accuracy
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:359d20c3e038cdbd408e2684c0f04fa2244837819d9c8dd2089e36e2a8e5d2ba
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
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spectral-conditional-molecule-generation

## Summary

Generate candidate 3D molecular structures from IR, Raman, and UV-Vis spectra by running DiffSpectra's spectrum-conditioned reverse diffusion (DMT denoiser guided by SpecFormer spectral embeddings), then score exact-match recovery against ground truth.

## When to use

Use when you have simulated or measured molecular spectra (IR, Raman, UV-Vis, or all jointly) and need de novo molecular structure elucidation evaluated by top-1/top-10 exact match rather than retrieval from a fixed library.

## When NOT to use

- You need to condition on NMR or mass-spectral data — the article states DiffSpectra currently focuses only on UV-Vis, IR, and Raman spectra
- The goal is retrieval of known structures from a fixed library rather than generative, de novo elucidation
- You only have a single spectrum type and expect joint-modality accuracy — single-modality runs (e.g. UV-Vis at ~0.3% top-1) substantially underperform

## Inputs

- QM9S test-set spectra (simulated IR, Raman, UV-Vis) in the DiffSpectra dataset format
- Trained DiffSpectra (DMT) checkpoint from huggingface.co/AzureLeon1/DiffSpectra (e.g. exp/pretrained_specformer.ckpt)
- configs/diffspectra_qm9s.py plus configs/base_qm9.py configuration files

## Outputs

- 10,000 generated 3D molecules per test spectrum (joint 2D topology + 3D coordinates)
- Per-experiment top-1 and top-10 exact-match accuracy and similarity metrics under exp/{exp_name}/eval/
- Ablation table comparing 'ir' vs 'raman' vs 'uv' vs 'allspectra' conditioning
- 3D structural RMSD metrics from eval_sampled_mols/rmsd.py

## How to apply

Install the environment from the repo's env.sh, download the QM9S dataset and the DMT checkpoint from HuggingFace, and set config.data.spectra_version to 'ir', 'raman', 'uv', or 'allspectra' (the last concatenates IR, Raman, and UV-Vis patch embeddings from SpecFormer as joint conditioning). Run main.py in eval mode with --config.eval.ckpts '40' and --config.eval.num_samples 10000 so each test spectrum yields 10,000 molecules via the reverse-time denoising process from t=1 to t=0. Top-1/top-10 exact-match accuracy and similarity metrics are computed automatically during generation, or afterwards via eval_sampled_mols/base_metrics.py; 3D quality is assessed with eval_sampled_mols/rmsd.py using Hungarian-matched RMSD. Expect joint multi-modal conditioning to outperform any single spectrum: reported ablation gives Raman ~30.3%, IR ~25.9%, UV-Vis ~0.3% top-1, and the full model reaches the headline 40.76% top-1 / 99.49% top-10.

## Related tools

- **DiffSpectra** (Spectra-conditioned diffusion framework executed in eval mode to generate molecules from spectra) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network producing joint 2D/3D molecular structures during reverse diffusion) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer spectral encoder whose patch embeddings (concatenated in the allspectra setting) condition the denoiser) — https://github.com/AzureLeon1/DiffSpectra
- **RDKit** (Used for molecule validation and canonical-structure comparison when computing exact-match accuracy)

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- The allspectra run reproduces the headline accuracy of 40.76% top-1 and 99.49% top-10
- The joint-modality top-1 accuracy exceeds the best single-spectrum setting (Raman, ~30.3%) by roughly +0.3% in the ablation table
- Each test spectrum yields exactly num_samples=10000 molecules and metrics land in exp/{exp_name}/eval/
- Exact-match metrics computed via eval_sampled_mols/base_metrics.py agree with the metrics emitted automatically during generation
- Single-modality sanity check: UV-Vis conditioning should give near-zero top-1 (~0.3%), far below Raman (~30.3%) and IR (~25.9%)

## Limitations

- Applicability to other modalities such as NMR or mass spectrometry remains unexplored; only UV-Vis, IR, and Raman are supported
- Validation is primarily computational — no experimental/laboratory confirmation of elucidated structures
- Results depend on simulated QM9S spectra; expanding the diversity and scale of spectral datasets is noted as needed for generalizability
- No software versions or dependency pins are stated beyond installing via env.sh

## Evidence

- [methods] The reverse-time denoising process starts from a standard Gaussian sample ... and proceeds backward from t=1 to t=0: [evidence span withheld — anchor status: not_found]
- [readme] --config.eval.num_samples 10000 \
    --config.eval.save_mols true \
    --config.data.spectra_version allspectra: "--config.eval.num_samples 10000 \
    --config.eval.save_mols true \
    --config.data.spectra_version allspectra"
