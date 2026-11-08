---
name: spectra-conditioned-molecular-diffusion-generation
description: Use when you need to generate/elucidate unknown molecular structures
  from multi-modal spectral data (IR, Raman, UV-Vis) as a conditional generation task
  rather than retrieval against a finite library.
when_to_use_negative:
- The spectral modality is outside the three supported types — the article notes applicability
  to NMR or mass spectrometry remains unexplored
- You only need unconditional molecular generation (tools like CDGS or JODO cover
  that); this skill is specifically spectra-conditioned elucidation
- You require laboratory/experimental validation of elucidated structures — the article
  states validation to date is primarily computational
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: 'The generative framework itself: conditional diffusion that produces molecular
    structures from multi-modal spectra'
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network parameterizing the joint 2D/3D diffusion
    over node, edge, and coordinate streams
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer spectral encoder (pre-trained with masked patches reconstruction
    and contrastive alignment) whose embeddings condition DMT denoising
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:254fa3edab0d3ebbcef0b9fde22d11d48fc934438c95aaed905480c8e1c000a5
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
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectra-conditioned-molecular-diffusion-generation

## Summary

Generates complete molecular structures (2D topology plus 3D coordinates) conditioned on multi-modal IR, Raman, and UV-Vis spectra via a conditional diffusion process, using the DiffSpectra framework with its DMT denoiser and SpecFormer spectral encoder.

## When to use

Use when you need to generate/elucidate unknown molecular structures from multi-modal spectral data (IR, Raman, UV-Vis) as a conditional generation task rather than retrieval against a finite library.

## When NOT to use

- The spectral modality is outside the three supported types — the article notes applicability to NMR or mass spectrometry remains unexplored
- You only need unconditional molecular generation (tools like CDGS or JODO cover that); this skill is specifically spectra-conditioned elucidation
- You require laboratory/experimental validation of elucidated structures — the article states validation to date is primarily computational

## Inputs

- Multi-modal spectra (IR, Raman, UV-Vis) from the QM9S test set (QM9 augmented with simulated spectra)
- Preprocessed QM9S dataset from huggingface.co/datasets/AzureLeon1/DiffSpectra
- Trained DMT/SpecFormer checkpoints from huggingface.co/AzureLeon1/DiffSpectra
- Configuration file configs/diffspectra_qm9s.py and base config configs/base_qm9.py

## Outputs

- Generated molecules with joint 2D topology (graph H, A) and 3D coordinates (X), saved under exp/{exp_name}/eval/
- Exact-structure accuracy (top-1, top-10) and similarity metrics (TaniSim_MG, TaniSim_MA, FGSim) versus ground-truth SMILES
- RMSD statistics after Hungarian atom-to-atom mapping against ground-truth 3D structures

## How to apply

Load the QM9S dataset (HuggingFace: AzureLeon1/DiffSpectra) and the DiffSpectra checkpoint, install the environment via env.sh, and point config.data.root to the dataset path. Configure configs/diffspectra_qm9s.py with --config.data.spectra_version allspectra and model.name DMT, supplying the pretrained SpecFormer checkpoint (masked-patch reconstruction + InfoNCE contrastive pre-training on QM9S) for best accuracy. For generation, run the reverse-time denoising process from t=1 to t=0 starting from a standard Gaussian sample, using the joint diffusion over node features H, edge features A, and 3D coordinates X parameterized by the SE(3)-equivariant Diffusion Molecule Transformer, guided by SpecFormer spectral embeddings. Sample 10,000 molecules per spectra condition with --config.eval.num_samples 10000 and --config.eval.ckpts '40'; accuracy and similarity metrics (top-1/top-10 exact structure, Tanimoto/functional-group similarities) are computed automatically and can be re-run via eval_sampled_mols/base_metrics.py. For 3D quality, align generated coordinates to ground truth via Hungarian atom mapping and compute RMSD with eval_sampled_mols/rmsd.py. The joint 2D+3D modality is essential: disabling the coordinate stream (2D-only) drops top-1 accuracy from 40.76% to 9.49%.

## Related tools

- **DiffSpectra** (The generative framework itself: conditional diffusion that produces molecular structures from multi-modal spectra) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network parameterizing the joint 2D/3D diffusion over node, edge, and coordinate streams) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer spectral encoder (pre-trained with masked patches reconstruction and contrastive alignment) whose embeddings condition DMT denoising) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.data.spectra_version allspectra
```

## Evaluation signals

- Joint 2D+3D run on QM9S test spectra should yield top-1 ≈ 40.76% and top-10 ≈ 99.49% exact-structure accuracy within diffusion sampling tolerance
- Ablating the 3D coordinate stream (2D-only) must show the reported drop to 9.49% top-1, confirming 3D geometric modeling benefits
- Similarity metrics should be in the reported ranges: TaniSim_MG ≈ 0.7875, TaniSim_MA ≈ 0.9262, FGSim ≈ 0.9652
- Generated molecules and metrics appear at exp/{exp_name}/eval/ after evaluation
- RMSD values must be computed only after Hungarian atom mapping against ground-truth 3D structures

## Limitations

- DiffSpectra currently focuses on three spectral types (UV-Vis, IR, Raman); NMR and MS remain unexplored
- Validation is primarily computational with no experimental/laboratory validation performed to date
- No software versions, model hyperparameters, or implementation details (diffusion framework / deep learning library) are stated beyond env.sh

## Evidence

- [readme] DiffSpectra achieves **40.76% top-1 accuracy** and **99.49% top-10 accuracy** in recovering exact molecular structures.: [evidence span withheld — anchor status: not_found]
