---
name: temporal-gradient-visualization-savitzky-golay-smoothing
description: Use when analyzing a 1,000-step diffusion denoising trajectory to reveal
  the temporal order in which spectral conditioning resolves atom semantics, simple
  substructures, and complex functional groups.
when_to_use_negative:
- The goal is end-to-end structure elucidation accuracy (top-1/top-10, RMSD) rather
  than interpretability of the denoising trajectory
- The model conditions on a single spectrum in isolation (ir/raman/uv-only configs)
  and the question concerns overall multi-modal conditioning rather than the temporal
  semantic-resolution order
- Input spectra come from modalities outside IR, Raman, and UV-Vis (e.g., NMR or MS),
  which DiffSpectra does not currently support
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: DiffSpectra / Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network whose per-step predicted atom-type outputs
    d_theta provide the gradient targets across the 1,000-step reverse process
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer spectral encoder whose input patch embeddings are the differentiation
    targets for the per-atom gradient computation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Savitzky-Golay filter
  role: Smoothing of the 20 averaged gradient-trend curves for visualization without
    distorting the four-phase temporal pattern
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:bc36d5f43eb817ab681cfb63b1d04d2d4be69e363f4a54ad9b2c14503fb3e0c6
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - DiffSpectra / Diffusion Molecule Transformer (DMT)
  - Savitzky-Golay filter
  - SpecFormer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# temporal-gradient-visualization-savitzky-golay-smoothing

## Summary

Visualizes how spectral information influences atom-type prediction across the DiffSpectra reverse denoising trajectory by computing per-atom gradients of predicted atom type d_theta w.r.t. SpecFormer spectral patches, averaging over 50-step windows (0.05 time intervals), and smoothing trends with a Savitzky-Golay filter.

## When to use

Use when analyzing a 1,000-step diffusion denoising trajectory to reveal the temporal order in which spectral conditioning resolves atom semantics, simple substructures, and complex functional groups.

## When NOT to use

- The goal is end-to-end structure elucidation accuracy (top-1/top-10, RMSD) rather than interpretability of the denoising trajectory
- The model conditions on a single spectrum in isolation (ir/raman/uv-only configs) and the question concerns overall multi-modal conditioning rather than the temporal semantic-resolution order
- Input spectra come from modalities outside IR, Raman, and UV-Vis (e.g., NMR or MS), which DiffSpectra does not currently support

## Inputs

- Trained DiffSpectra DMT model checkpoint (e.g., exp/pretrained_specformer.ckpt and eval ckpts from huggingface.co/AzureLeon1/DiffSpectra)
- Molecular spectra sample (IR, Raman, UV-Vis) from the QM9S dataset (huggingface.co/datasets/AzureLeon1/DiffSpectra)
- SpecFormer spectral patch embeddings from the forward pass

## Outputs

- gradient_heatmaps_50step.npy — array of shape (20, N_atoms, num_patches) of 50-step-averaged gradient heatmaps
- gradient_trajectory_summary.csv — per-time-interval gradient statistics (20 rows, one per 0.05 interval)
- Smoothed temporal trend plots of per-atom gradient magnitudes

## How to apply

Load the trained DiffSpectra DMT checkpoint (available on Hugging Face at huggingface.co/AzureLeon1/DiffSpectra) and run the reverse denoising process (1,000 steps, t from 1 to 0) on a QM9S spectra sample. At each step, backpropagate the gradient of the predicted atom-type d_theta output for each atom with respect to the input spectral patch embeddings from SpecFormer, and stack the per-step heatmaps into a (1000, N_atoms, num_patches) tensor. Average every 50 consecutive steps — corresponding to 0.05 intervals along the generative time axis — yielding 20 averaged slices, then optionally smooth the temporal trends with a Savitzky-Golay filter (Steinier et al., 1964) for visualization. Save results as gradient_heatmaps_50step.npy plus a gradient_trajectory_summary.csv of per-time-interval statistics, and judge correctness by whether the four-phase pattern (elevated gradients t~1.0-0.7 for all atoms; simple C-C/C-H substructures in 0.7-0.3; complex groups like ethers, hydroxyls, alkynes in 0.3-0.1; diminishing gradients in 0.1-0.0) is reproduced.

## Related tools

- **DiffSpectra / Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network whose per-step predicted atom-type outputs d_theta provide the gradient targets across the 1,000-step reverse process) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer spectral encoder whose input patch embeddings are the differentiation targets for the per-atom gradient computation) — https://github.com/AzureLeon1/DiffSpectra
- **Savitzky-Golay filter** (Smoothing of the 20 averaged gradient-trend curves for visualization without distorting the four-phase temporal pattern)

## Evaluation signals

- Output tensor gradient_heatmaps_50step.npy exists, is finite-valued, and has shape (20, N_atoms, num_patches)
- The temporal trajectory matches the reported pattern: elevated gradients for all atoms in t~1.0-0.7, stronger gradients on simple C-C/C-H substructures in 0.7-0.3, preferential resolution of ethers/hydroxyls/alkynes in 0.3-0.1, and diminishing gradients in 0.1-0.0 (expert review of heatmap slices)
- gradient_trajectory_summary.csv contains exactly 20 rows, one per 0.05 time interval
- Savitzky-Golay smoothing preserves the phase boundaries and peak locations of the raw averaged curves rather than flattening the transitions
- Per-step gradients are well-defined (backprop succeeds through both DMT and the SpecFormer patch embeddings at every one of the 1,000 steps)

## Limitations

- Validation is primarily computational; no experimental/laboratory validation has been performed
- Gradient trajectories were characterized on QM9S (QM9 augmented with simulated spectra), so patterns may not transfer to experimental spectra or larger molecules
- DiffSpectra covers only three spectral types (UV-Vis, IR, Raman); gradients cannot be computed for NMR or MS modalities
- No software versions or implementation details (diffusion framework / deep learning library) are documented, complicating reproduction of the gradient analysis

## Evidence
