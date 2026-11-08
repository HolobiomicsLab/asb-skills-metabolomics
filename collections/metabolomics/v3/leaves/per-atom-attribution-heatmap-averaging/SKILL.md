---
name: per-atom-attribution-heatmap-averaging
description: Use when you have a trained diffusion-based generative model over molecules
  and spectra, and you want to attribute which spectral patches influence which atom-type
  decisions at each stage of the denoising trajectory.
when_to_use_negative:
- The generative model does not expose an iterative denoising trajectory (e.g., single-pass
  autoregressive SMILES decoders), so there is no time axis over which to average
  attribution heatmaps.
- The spectral modalities are outside DiffSpectra's supported set (IR, Raman, UV-Vis)
  — e.g., NMR or mass spectrometry — since the paper states applicability to those
  modalities remains unexplored.
- The goal is a single post-hoc saliency map for a final prediction rather than characterizing
  the temporal order of structural resolution across diffusion phases.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: DiffSpectra / Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network whose predicted atom-type output d_theta
    is differentiated with respect to spectral patch inputs at each reverse-diffusion
    step
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder whose patch embeddings serve as the input
    side of the gradient computation, linking spectra to atom-type predictions
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Savitzky-Golay filter
  role: Optional smoothing of the 50-step-averaged gradient trends along the generative
    time axis for visualization
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:14c61b1bc803a0093ea80a4dbc441310c566867f4481a6d8fbfacab35ffb876c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Per-atom attribution heatmap averaging

## Summary

Computes per-atom gradients of the predicted atom type with respect to spectral patch embeddings at each reverse-diffusion step, then averages the resulting heatmaps over 50-step windows (Δt = 0.05) to trace how spectral influence drives structure elucidation across the generative trajectory.

## When to use

Use when you have a trained diffusion-based generative model over molecules and spectra, and you want to attribute which spectral patches influence which atom-type decisions at each stage of the denoising trajectory.

## When NOT to use

- The generative model does not expose an iterative denoising trajectory (e.g., single-pass autoregressive SMILES decoders), so there is no time axis over which to average attribution heatmaps.
- The spectral modalities are outside DiffSpectra's supported set (IR, Raman, UV-Vis) — e.g., NMR or mass spectrometry — since the paper states applicability to those modalities remains unexplored.
- The goal is a single post-hoc saliency map for a final prediction rather than characterizing the temporal order of structural resolution across diffusion phases.

## Inputs

- Trained DiffSpectra DMT model checkpoint (e.g., pretrained_specformer.ckpt-based allspectra run)
- Multi-modal spectral sample (IR, Raman, UV-Vis) from the QM9S dataset, segmented into patches
- Reverse denoising trajectory with 1,000 steps over continuous time t=1 to t=0

## Outputs

- Gradient heatmap tensor gradient_heatmaps_50step.npy of shape (20, N_atoms, num_patches)
- Summary table gradient_trajectory_summary.csv with per-time-interval statistics
- Optionally, Savitzky-Golay-smoothed temporal gradient trend curves for visualization

## How to apply

Load the trained DiffSpectra DMT checkpoint (e.g., from the huggingface.co/AzureLeon1/DiffSpectra checkpoints) and a multi-modal spectral sample (IR, Raman, UV-Vis) from the QM9S dataset. Run the full 1,000-step reverse denoising process from t=1 to t=0, and at each step backpropagate the gradient of the model's predicted atom type output d_theta for each atom with respect to the spectral patch embeddings produced by SpecFormer. Stack the per-step per-atom/per-patch gradients into a tensor of shape (1000, N_atoms, num_patches), then average over every 50 consecutive steps — corresponding to time intervals of 0.05 along the generative time axis — yielding 20 averaged heatmap slices. Optionally smooth the averaged temporal trends with a Savitzky-Golay filter for visualization. Interpret the trajectory as phase-structured: expect elevated gradients for all atoms early (t~1.0–0.7), preferential gradients on simple substructures (C-C, C-H) in the early phase (0.7–0.3), stronger gradients on complex functional groups (ethers, hydroxyls, alkynes) in the late phase (0.3–0.1), and diminishing gradients as elucidation converges (0.1–0.0).

## Related tools

- **DiffSpectra / Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network whose predicted atom-type output d_theta is differentiated with respect to spectral patch inputs at each reverse-diffusion step) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder whose patch embeddings serve as the input side of the gradient computation, linking spectra to atom-type predictions) — https://github.com/AzureLeon1/DiffSpectra
- **Savitzky-Golay filter** (Optional smoothing of the 50-step-averaged gradient trends along the generative time axis for visualization)

## Evaluation signals

- Output tensor gradient_heatmaps_50step.npy exists and has shape (20, N_atoms, num_patches), i.e., 20 windows of 50 denoising steps each from the 1,000-step trajectory.
- All values in the averaged heatmap tensor are finite (no NaN/Inf from backpropagation through the DMT network).
- The temporal trajectory of semantic information matches the reported pattern: broad atom-level semantics first, then simple substructures (C-C, C-H), then complex functional groups (ethers, hydroxyls, alkynes).
- Gradients diminish in the final phase (t = 0.1–0.0), consistent with convergence of the elucidation process.
- Averaged windows correspond to time intervals of exactly 0.05 along the generative time axis (1,000 steps / 50-step windows).

## Limitations

- Validation of the phase pattern is primarily computational and relies on expert review of heatmap slices; no experimental/laboratory validation was performed.
- The analysis is demonstrated on QM9S (QM9 augmented with simulated spectra) with three spectral types only; NMR and mass spectrometry modalities remain unexplored.
- Averaging over 50-step windows (Δt = 0.05) is a coarse aggregation that may obscure step-level attribution dynamics within each phase.

## Evidence

- [methods] reverse denoising process definition: [evidence span withheld — anchor status: not_found]
