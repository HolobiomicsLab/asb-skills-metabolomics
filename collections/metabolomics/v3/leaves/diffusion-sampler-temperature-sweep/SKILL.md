---
name: diffusion-sampler-temperature-sweep
description: Decision support for choosing the noise-scaling temperature τ when generating
  candidate molecules with a spectra-conditioned diffusion sampler, especially when
  generated sets look overly deterministic (low τ) or too random (high τ).
when_to_use_negative:
- The task is training SpecFormer or the diffusion model itself rather than tuning
  sampling-time noise scale.
- Only a single fixed temperature is required and no sweep/diversity-vs-accuracy comparison
  is needed.
- Spectra are NMR or mass spectrometry modalities, which DiffSpectra explicitly does
  not support.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: DiffSpectra
  role: Generative diffusion framework whose reverse-time sampler is being temperature-swept
    to produce molecules from spectra
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network parameterizing the sampler whose noise
    scaling Gₛ is modulated by τ
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder providing conditional guidance during each denoising step
    at every τ
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:a5ee7b4b8a68b990a041e5f8f9a86b863e4587295db8468bb8bd8e29a2353c22
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# diffusion-sampler-temperature-sweep

## Summary

Sweeps the sampling temperature τ that scales injected noise during DiffSpectra's reverse-time denoising to map the trade-off between structural diversity and elucidation accuracy, with τ=1.0 as the best setting (40.76% top-1 accuracy).

## When to use

Decision support for choosing the noise-scaling temperature τ when generating candidate molecules with a spectra-conditioned diffusion sampler, especially when generated sets look overly deterministic (low τ) or too random (high τ).

## When NOT to use

- The task is training SpecFormer or the diffusion model itself rather than tuning sampling-time noise scale.
- Only a single fixed temperature is required and no sweep/diversity-vs-accuracy comparison is needed.
- Spectra are NMR or mass spectrometry modalities, which DiffSpectra explicitly does not support.

## Inputs

- QM9S test spectra (IR, Raman, UV-Vis modalities)
- Trained DiffSpectra / DMT checkpoint (e.g. exp/allspectra_pretrained_specformer, eval ckpts '40')
- Temperature grid values τ (e.g. 0.0–2.0)
- Ground-truth QM9S reference structures for accuracy/diversity metrics

## Outputs

- Per-τ generated molecule sets (e.g. 10,000 samples each) saved under exp/{exp_name}/eval/
- Temperature sweep table (tau, top1_accuracy, top10_accuracy, unique_atom_types, unique_bond_types, unique_smiles)
- Chosen optimal τ with justification

## How to apply

For each τ in a grid such as {0.0, 0.2, 0.4, 0.6, 0.8, 1.0, 1.2, 1.5, 2.0}, run the reverse-time denoising process of the DiffSpectra (DMT) model, starting from a standard Gaussian sample 𝒞₁ ~ N(0, I) and scaling noise as Gₛ = Ḡₛ + τ · (σₛ σₜ|ₛ / σₜ) G∈, sampling ~10,000 molecules per setting from QM9S test spectra. Compute elucidation metrics (top-1 and top-10 atom type / bond type accuracy) and diversity metrics (unique atom types, unique bond types, unique SMILES) per τ. Aggregate into a table sorted by τ; in the article, τ=1.0 gave the best trade-off with 40.76% Acc@1, τ=0.0 degraded accuracy (14.65%) with high MCES (4.6948), and τ=1.2 also degraded (39.59%) due to excessive randomness.

## Related tools

- **DiffSpectra** (Generative diffusion framework whose reverse-time sampler is being temperature-swept to produce molecules from spectra) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network parameterizing the sampler whose noise scaling Gₛ is modulated by τ) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder providing conditional guidance during each denoising step at every τ) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.data.spectra_version allspectra
```

## Evaluation signals

- Entry at τ=1.0 reproduces top-1 accuracy of 40.76%; τ=0.0 and τ=1.2 both report 39.59% in the ablation trend.
- Table columns must include tau, top1_accuracy, top10_accuracy, unique_atom_types, unique_bond_types, unique_smiles, sorted ascending by τ.
- MCES at τ=0.0 rises to 4.6948 while τ=0.0 accuracy drops to 14.65%, reflecting overly deterministic sampling.
- Sampling count per temperature should be 10,000 molecules to match the reported evaluation protocol.

## Limitations

- Framework is limited to three spectral types (UV-Vis, IR, Raman); NMR or MS is not applicable.
- Validation was primarily computational; no experimental laboratory confirmation.
- τ=0.0 produces overly deterministic results with reduced accuracy and elevated MCES; τ=1.2 degrades due to excessive randomness.
- Checkpoints and environment must be installed per env.sh and configured in configs/ before running.

## Evidence

- [methods] reverse-time denoising process starting from a standard Gaussian sample: [evidence span withheld — anchor status: not_found]
- [readme] noise scaling during sampling: [evidence span withheld — anchor status: not_found]
- [results] best temperature setting: [evidence span withheld — anchor status: not_found]
- [readme] evaluation script: [evidence span withheld — anchor status: not_found]
- [results] ablation findings: [evidence span withheld — anchor status: not_found]
