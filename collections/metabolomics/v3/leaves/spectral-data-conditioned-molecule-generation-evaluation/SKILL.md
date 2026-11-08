---
name: spectral-data-conditioned-molecule-generation-evaluation
description: Use when a checkpoint of a spectra-conditioned molecular generator (e.g.
  DiffSpectra with SpecFormer + DMT) must be benchmarked on spectral input, producing
  accuracy, FCD and RMSD metrics comparable to paper ablations.
when_to_use_negative:
- When the goal is unconditional molecular generation benchmarking — DiffSpectra's
  pipeline scores exact structure recovery conditioned on spectra, not distribution-level
  generative quality alone
- When input spectra are single-modality NMR or mass spectrometry — the framework
  is limited to UV-Vis, IR, and Raman modalities
- When only qualitative inspection of generated molecules is needed without ground-truth
  matching — the accuracy/FCD metrics require QM9S ground-truth structures
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_2269
tools:
- name: DiffSpectra
  role: Generative framework evaluated in eval mode; generates molecules conditioned
    on multi-modal spectra via diffusion
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder whose pre-trained weights (specformer.ckpt) provide the spectral
    embeddings guiding denoising; the ablation swaps it for random initialization
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network whose sampled structures are scored by
    the evaluation scripts
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:88afdda5236f7be00da1a004ce10b93929b008cefd76fee229fed0dd5d243257
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spectral-data-conditioned-molecule-generation-evaluation

## Summary

Evaluates spectra-conditioned generative diffusion models (DiffSpectra-style) by generating molecules from multi-modal spectra (IR, Raman, UV-Vis) and scoring top-1/top-10 exact-match accuracy, Fréchet ChemNet Distance (FCD), and RMSD against ground-truth 3D structures.

## When to use

Use when a checkpoint of a spectra-conditioned molecular generator (e.g. DiffSpectra with SpecFormer + DMT) must be benchmarked on spectral input, producing accuracy, FCD and RMSD metrics comparable to paper ablations.

## When NOT to use

- When the goal is unconditional molecular generation benchmarking — DiffSpectra's pipeline scores exact structure recovery conditioned on spectra, not distribution-level generative quality alone
- When input spectra are single-modality NMR or mass spectrometry — the framework is limited to UV-Vis, IR, and Raman modalities
- When only qualitative inspection of generated molecules is needed without ground-truth matching — the accuracy/FCD metrics require QM9S ground-truth structures

## Inputs

- Trained DiffSpectra model checkpoints (e.g. exp/pretrained_specformer.ckpt or randomly initialized SpecFormer variants) from huggingface.co/AzureLeon1/DiffSpectra
- QM9S test-set multi-modal spectra (IR, Raman, UV-Vis) from the Hugging Face dataset at huggingface.co/datasets/AzureLeon1/DiffSpectra
- Configuration files configs/diffspectra_qm9s.py and configs/base_qm9.py with data.root pointing to the local dataset path
- Ground-truth 2D structures (SMILES) and 3D conformations for matching and RMSD computation

## Outputs

- Generated molecules (saved SMILES/structures) under exp/{exp_name}/eval/
- Top-1 and top-10 structure elucidation accuracy (expected 40.76% / 99.49% for the full model on QM9S)
- Fréchet ChemNet Distance (FCD) values
- Per-molecule RMSD distributions against ground-truth 3D structures
- Ablation comparison tables (e.g. specformer_ablation_metrics.csv with pre-trained vs random SpecFormer columns)

## How to apply

Run the model in 'eval' mode (main.py --mode eval) with --config.eval.num_samples 10000 and --config.eval.save_mols true, selecting the spectra_version (allspectra, ir, raman, uv) to match the conditioning regime under test; generation and metric computation happen together, and molecules are matched to ground truth by SMILES identity. Accuracy and similarity metrics come from base_metrics.py; 3D structural fidelity comes from rmsd.py, which uses Hungarian-algorithm atom mapping to compute RMSD against ground-truth conformations. Results land in exp/{exp_name}/eval/. For ablation comparisons (e.g. pre-trained vs random SpecFormer), run both configurations identically and check deltas — the paper reports top-1 = 40.76% with pre-trained SpecFormer vs 35.64% without (Δ ≈ +9.49 pp) and an FCD delta of 1.651.

## Related tools

- **DiffSpectra** (Generative framework evaluated in eval mode; generates molecules conditioned on multi-modal spectra via diffusion) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder whose pre-trained weights (specformer.ckpt) provide the spectral embeddings guiding denoising; the ablation swaps it for random initialization) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network whose sampled structures are scored by the evaluation scripts) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Top-1 accuracy for the full pre-trained configuration should reach ≈40.76% and top-10 ≈99.49% on the 10,000-sample QM9S evaluation
- Pre-trained vs untrained SpecFormer delta in top-1 accuracy should reproduce ≈+9.49 percentage points (40.76% vs 35.64%)
- FCD delta between the two SpecFormer variants should match the reported 1.651
- Generated molecule files and metrics must appear under exp/{exp_name}/eval/ after base_metrics.py / rmsd.py run
- SMILES-identity matching between generated and ground-truth molecules must be exact for top-k accuracy counting

## Limitations

- Validation is primarily computational; no experimental/laboratory validation has been performed
- Framework covers only three spectral types (UV-Vis, IR, Raman); applicability to NMR or MS is unexplored
- No software versions, model hyperparameters, or deep-learning library details are stated beyond the env.sh installation script
- Metrics depend on simulated spectra from QM9S (QM9 augmented with simulated spectra); real experimental spectra may behave differently

## Evidence
