---
name: spectrum-conditioned-denoising-trajectory-analysis
description: Use when you need to expose the elucidation order inside a diffusion-based
  structure generator, e.g. testing whether broad atom semantics are established before
  simple bonds and then complex functional groups.
when_to_use_negative:
- When the goal is end-task accuracy evaluation of generated structures (top-1/top-10
  accuracy, Hungarian-mapped RMSD) rather than mechanistic trajectory interpretation
- When the generative model lacks a differentiable, patch-conditioned spectral encoder
  (e.g. SMILES-based autoregressive models that ignore 3D geometry and spectra)
- When you need per-functional-group presence/absence classification performance (that
  calls for the ROC-AUC-per-group protocol, not gradient trajectory analysis)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: Framework whose trained checkpoint supplies the 1,000-step reverse denoising
    process over which per-atom gradients are computed
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network whose predicted atom type output d_theta
    is differentiated at each denoising step
- name: SpecFormer
  role: Spectral encoder producing the input patch embeddings that serve as the differentiation
    target of the per-atom gradients
- name: Savitzky-Golay filter
  role: Optional smoothing of the 50-step-averaged gradient trends for visualization
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:37c893f72098bdd6eaed24a84284cd0b5d60968848988ca91f9d67171a7f4d1d
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Diffusion Molecule Transformer (DMT)
  - Savitzky-Golay filter
  - SpecFormer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectrum-conditioned-denoising-trajectory-analysis

## Summary

A mechanistic-analysis skill for spectra-conditioned diffusion models that traces when spectral information resolves atom-level semantics, simple substructures, and complex functional groups during generation. It does this by backpropagating per-atom gradients of the predicted atom type d_theta output with respect to spectral patch embeddings across the 1,000-step reverse denoising trajectory.

## When to use

Use when you need to expose the elucidation order inside a diffusion-based structure generator, e.g. testing whether broad atom semantics are established before simple bonds and then complex functional groups.

## When NOT to use

- When the goal is end-task accuracy evaluation of generated structures (top-1/top-10 accuracy, Hungarian-mapped RMSD) rather than mechanistic trajectory interpretation
- When the generative model lacks a differentiable, patch-conditioned spectral encoder (e.g. SMILES-based autoregressive models that ignore 3D geometry and spectra)
- When you need per-functional-group presence/absence classification performance (that calls for the ROC-AUC-per-group protocol, not gradient trajectory analysis)

## Inputs

- Trained DiffSpectra DMT checkpoint (e.g. from huggingface.co/AzureLeon1/DiffSpectra, placed in exp/)
- Molecular sample with multi-modal spectra (IR, Raman, UV-Vis) from the QM9S dataset
- SpecFormer spectral patch embeddings (patches segmented from each spectrum type)

## Outputs

- Gradient heatmap tensor gradient_heatmaps_50step.npy of shape (20, N_atoms, num_patches)
- Summary table gradient_trajectory_summary.csv with per-time-interval gradient statistics
- Qualitative phase characterization: semantic establishment -> simple substructures -> complex functional groups -> convergence

## How to apply

Load the trained DiffSpectra Diffusion Molecule Transformer (DMT) checkpoint and a molecular spectra sample (IR, Raman, UV-Vis) from the QM9S dataset, then run the reverse-time denoising process over 1,000 steps from t=1 to t=0. At each step t, backpropagate the gradient of the predicted atom type d_theta output for each atom with respect to the input spectral patch embeddings produced by SpecFormer, and stack the per-step gradient heatmaps into a tensor of shape (1000, N_atoms, num_patches). Average the heatmaps over every 50 consecutive steps (time intervals of 0.05 along the generative time axis) to yield 20 averaged slices, optionally smoothing the trends with a Savitzky-Golay filter for visualization. Save the averaged heatmaps as gradient_heatmaps_50step.npy and a per-time-interval statistics table gradient_trajectory_summary.csv. Interpret the trajectory against the four reported phases: elevated gradients for all atoms at t~1.0-0.7, preferential gradients on simple substructures (C-C, C-H bonds) at 0.7-0.3, preferential resolution of complex groups (ethers, hydroxyls, alkynes) at 0.3-0.1, and diminishing gradients as elucidation converges at 0.1-0.0.

## Related tools

- **DiffSpectra** (Framework whose trained checkpoint supplies the 1,000-step reverse denoising process over which per-atom gradients are computed) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network whose predicted atom type output d_theta is differentiated at each denoising step)
- **SpecFormer** (Spectral encoder producing the input patch embeddings that serve as the differentiation target of the per-atom gradients)
- **Savitzky-Golay filter** (Optional smoothing of the 50-step-averaged gradient trends for visualization)

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.data.spectra_version allspectra
```

## Evaluation signals

- gradient_heatmaps_50step.npy exists and has shape (20, N_atoms, num_patches)
- All entries of the averaged gradient tensor are finite (no NaN/Inf from the 1,000 backpropagations)
- The temporal trajectory of semantic information matches the reported pattern: global atom-type semantics first, then simple substructures, then complex functional groups
- The four phase boundaries (t~1.0-0.7, 0.7-0.3, 0.3-0.1, 0.1-0.0) are distinguishable in the 20 averaged heatmap slices
- gradient_trajectory_summary.csv contains one statistics row per 50-step time interval (20 rows)

## Limitations

- Analysis is tied to the DMT/SpecFormer architecture and its spectral patch conditioning; it does not transfer to non-diffusion or non-spectra-conditioned generators
- DiffSpectra currently covers only three spectral types (UV-Vis, IR, Raman); applicability to NMR or mass spectrometry remains unexplored
- Validation of the phase interpretation is primarily computational and relies on expert review of heatmap slices; no experimental validation was performed

## Evidence
