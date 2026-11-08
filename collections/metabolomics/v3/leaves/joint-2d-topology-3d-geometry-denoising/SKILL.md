---
name: joint-2d-topology-3d-geometry-denoising
description: Use when you need to quantify how much jointly generating 3D coordinates
  (versus 2D-topological-only diffusion) improves spectra-conditioned exact-structure
  elucidation, using metrics like top-1/top-10 accuracy, TaniSim_MG/MA, FGSim, MCES,
  and RMSD.
when_to_use_negative:
- The goal is unconditional or non-spectra-conditioned molecule generation — DiffSpectra's
  generation here is driven by QM9S spectral conditioning, not a molecular prior
- Your spectra fall outside the three modalities DiffSpectra covers (UV-Vis, IR, Raman)
  — the discussion notes applicability to NMR or MS remains unexplored
- You lack trained checkpoints for both the joint and 2D-only models, since the ablation
  requires re-running an otherwise-identical sampler with the coordinate stream disabled
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: Spectra-conditioned diffusion framework whose sampler is run in 'allspectra'
    mode for the joint 2D+3D variant and with the coordinate stream disabled for the
    2D-only ablation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network (data prediction model d_theta with self-conditioning)
    that models node, edge, and coordinate streams during reverse-time denoising
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder providing embeddings for IR/Raman/UV-Vis
    patches that condition the denoising process
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:51ea2f959fff42ab2fc25468977f039802b177f7d38b9c67b782c6bc53fc2c47
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
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Joint 2D Topology & 3D Geometry Denoising for Molecular Structure Elucidation

## Summary

Runs a conditional reverse-time diffusion process (DMT, SE(3)-equivariant) that simultaneously denoises node features H, edge features A, and 3D coordinates X to elucidate molecules from IR/Raman/UV-Vis spectra. This skill covers generating with the joint 2D+3D stream and ablated 2D-only variant, then benchmarking the accuracy gap between them.

## When to use

Use when you need to quantify how much jointly generating 3D coordinates (versus 2D-topological-only diffusion) improves spectra-conditioned exact-structure elucidation, using metrics like top-1/top-10 accuracy, TaniSim_MG/MA, FGSim, MCES, and RMSD.

## When NOT to use

- The goal is unconditional or non-spectra-conditioned molecule generation — DiffSpectra's generation here is driven by QM9S spectral conditioning, not a molecular prior
- Your spectra fall outside the three modalities DiffSpectra covers (UV-Vis, IR, Raman) — the discussion notes applicability to NMR or MS remains unexplored
- You lack trained checkpoints for both the joint and 2D-only models, since the ablation requires re-running an otherwise-identical sampler with the coordinate stream disabled

## Inputs

- QM9S test-split spectra (IR, Raman, UV-Vis) from huggingface.co/datasets/AzureLeon1/DiffSpectra
- Pretrained DiffSpectra/DMT checkpoint from huggingface.co/AzureLeon1/DiffSpectra
- Configuration file configs/diffspectra_qm9s.py with data.root set to the dataset path
- Ground-truth molecular structures (SMILES and 3D coordinates) for the QM9S test set

## Outputs

- Generated molecules (2D graphs + 3D coordinates) for the joint and 2D-only variants, saved under exp/{exp_name}/eval/
- Exact-structure top-1 and top-10 accuracy per variant
- Fingerprint/fragment/functional-group similarity metrics (TaniSim_MG, TaniSim_MA, FGSim, MCES) per variant
- RMSD statistics after Hungarian atom-to-atom mapping for the joint 2D+3D variant
- Aggregated CSV table with one row per variant and columns for accuracies, similarities, and RMSD

## How to apply

Load the QM9S dataset (HuggingFace AzureLeon1/DiffSpectra) and the trained DiffSpectra checkpoint, and set data.spectra_version to 'allspectra' (IR + Raman + UV-Vis conditioning). Run the eval-mode sampler with --config.eval.ckpts 40 and --config.eval.num_samples 10000, executing reverse-time denoising from t=1 to t=0 starting from a standard Gaussian sample, with the full joint diffusion over H, A, and X via the data-prediction model d_theta with self-conditioning. Re-run the identical sampler with the coordinate stream X disabled (2D-only diffusion over H and A) to obtain the ablated variant. Compare generated SMILES against ground-truth structures for exact top-1/top-10 accuracy and fingerprint similarities (TaniSim_MG, TaniSim_MA, FGSim, MCES), mirroring eval_sampled_mols/base_metrics.py; for the joint variant, additionally run eval_sampled_mols/rmsd.py to align generated 3D coordinates to ground truth via Hungarian atom mapping and record RMSD. Aggregate both variants into one table; the expected gap is top-1 9.49% (2D-only) versus 40.76% (joint 2D+3D) and top-10 ≈99.49%, with similarity gains such as TaniSim_MG 0.5609→0.7875 and FGSim 0.9034→0.9652.

## Related tools

- **DiffSpectra** (Spectra-conditioned diffusion framework whose sampler is run in 'allspectra' mode for the joint 2D+3D variant and with the coordinate stream disabled for the 2D-only ablation) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network (data prediction model d_theta with self-conditioning) that models node, edge, and coordinate streams during reverse-time denoising) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder providing embeddings for IR/Raman/UV-Vis patches that condition the denoising process)

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --mode eval --workdir exp/allspectra --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.data.spectra_version allspectra
```

## Evaluation signals

- Joint 2D+3D run must reproduce reported top-1 ≈40.76% and top-10 ≈99.49% within run-to-run diffusion sampling tolerance
- 2D-only ablation must show the reported drop to top-1 ≈9.49%, confirming that 3D geometric modeling benefits performance
- Similarity metrics for the joint variant should exceed the 2D-only values in the reported directions (TaniSim_MG 0.5609→0.7875, TaniSim_MA 0.8350→0.9262, FGSim 0.9034→0.9652)
- RMSD for the joint variant must be computed only after Hungarian atom-to-atom mapping against ground-truth 3D structures
- Generated molecules and metrics must be present under exp/{exp_name}/eval/ for both variants

## Limitations

- Diffusion sampling is stochastic, so exact top-1/top-10 accuracies carry run-to-run tolerance rather than bit-exact reproduction
- Validation is primarily computational — no experimental/laboratory validation has been performed
- The QM9S dataset augments QM9 with simulated spectra, so absolute accuracy may not transfer to experimental spectra

## Evidence

- [results] ablation finding: 2D-only vs joint accuracy: [evidence span withheld — anchor status: not_found]
- [readme] manual metric computation scripts: [evidence span withheld — anchor status: not_found]
