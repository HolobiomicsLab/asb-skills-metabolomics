---
name: multi-modal-spectra-ablation-analysis
description: Use when you need to isolate the contribution of each spectral modality
  (IR, Raman, UV-Vis) to a conditional generative model's structure-elucidation accuracy,
  e.g. to reproduce or extend the DiffSpectra modality ablation table.
when_to_use_negative:
- The question is about 3D structural fidelity rather than conditioning-modality effects
  — use the RMSD/Hungarian-atom-mapping evaluation path (eval_sampled_mols/rmsd.py)
  instead of the accuracy ablation.
- The spectral modalities of interest are NMR or mass spectrometry — the article states
  DiffSpectra 'currently focuses on three spectral types—UV-Vis, IR, and Raman' and
  applicability to other modalities remains unexplored.
- You want general unconditional molecular-generation metrics (e.g. comparison against
  CDGS or JODO baselines) rather than spectrum-conditioned structure elucidation accuracy.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3315
tools:
- name: DiffSpectra
  role: Spectra-conditioned diffusion framework whose DMT denoiser is evaluated under
    each modality setting (spectra_version ir/raman/uv/allspectra).
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network that performs the conditional reverse-time
    generation of candidate molecules in each ablation arm.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder whose IR/Raman/UV-Vis patch embeddings
    are concatenated as joint conditioning in the multi-modal arm; pre-trained checkpoint
    supplied via config.model.pretrained_specformer_path.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: base_metrics.py
  role: Repository script (eval_sampled_mols/base_metrics.py) that computes exact-match
    top-1/top-10 accuracy and similarity metrics on generated molecules, either automatically
    or when run manually.
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:cb6628af907832bbc07060855484420599319305310d4956c6f7895c01616f11
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
  - base_metrics.py
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# multi-modal-spectra-ablation-analysis

## Summary

Runs a controlled conditioning-modality ablation for a spectra-conditioned diffusion generator (DiffSpectra/DMT), comparing IR-only, Raman-only, UV-Vis-only and joint 'allspectra' conditioning on the QM9S test set to quantify how much multi-modal spectral input improves structure elucidation accuracy.

## When to use

Use when you need to isolate the contribution of each spectral modality (IR, Raman, UV-Vis) to a conditional generative model's structure-elucidation accuracy, e.g. to reproduce or extend the DiffSpectra modality ablation table.

## When NOT to use

- The question is about 3D structural fidelity rather than conditioning-modality effects — use the RMSD/Hungarian-atom-mapping evaluation path (eval_sampled_mols/rmsd.py) instead of the accuracy ablation.
- The spectral modalities of interest are NMR or mass spectrometry — the article states DiffSpectra 'currently focuses on three spectral types—UV-Vis, IR, and Raman' and applicability to other modalities remains unexplored.
- You want general unconditional molecular-generation metrics (e.g. comparison against CDGS or JODO baselines) rather than spectrum-conditioned structure elucidation accuracy.

## Inputs

- QM9S test set with simulated spectra (IR, Raman, UV-Vis) per molecule, from the HuggingFace dataset repo
- Trained DiffSpectra (DMT) checkpoints from huggingface.co/AzureLeon1/DiffSpectra (including the pretrained-SpecFormer variant for allspectra)
- Configuration files configs/diffspectra_qm9s.py and configs/base_qm9.py

## Outputs

- Per-modality top-1 and top-10 exact-match accuracy numbers for ir / raman / uv / allspectra conditions
- Generated molecules saved under exp/{exp_name}/eval/ (config.eval.save_mols true) for each modality setting
- Aggregated four-row modality ablation table

## How to apply

For each condition, run DiffSpectra (DMT backbone) in eval mode with `--config.data.spectra_version` set to 'ir', 'raman', 'uv', or 'allspectra', loading the trained checkpoint (eval.ckpts '40') and sampling 10,000 molecules per spectrum via the reverse-time denoising process from t=1 to t=0. In the 'allspectra' condition, SpecFormer patch embeddings for IR, Raman, and UV-Vis are jointly used as conditioning. Exact-match top-1/top-10 accuracy against ground-truth structures is computed automatically during generation or manually with `eval_sampled_mols/base_metrics.py`. Judge the result by (a) whether the allspectra run reproduces the reported 40.76% top-1 / 99.49% top-10 headline accuracy, and (b) whether joint conditioning beats the best single-modality setting (Raman) by roughly +0.3% or more on top-1.

## Related tools

- **DiffSpectra** (Spectra-conditioned diffusion framework whose DMT denoiser is evaluated under each modality setting (spectra_version ir/raman/uv/allspectra).) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network that performs the conditional reverse-time generation of candidate molecules in each ablation arm.) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder whose IR/Raman/UV-Vis patch embeddings are concatenated as joint conditioning in the multi-modal arm; pre-trained checkpoint supplied via config.model.pretrained_specformer_path.) — https://github.com/AzureLeon1/DiffSpectra
- **base_metrics.py** (Repository script (eval_sampled_mols/base_metrics.py) that computes exact-match top-1/top-10 accuracy and similarity metrics on generated molecules, either automatically or when run manually.) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- The 'allspectra' condition reproduces the reported headline accuracy of 40.76% top-1 and 99.49% top-10 exact-match accuracy.
- Top-1 accuracy ordering across arms is allspectra > Raman (30.3%) > IR (25.9%) > UV-Vis (0.3%), with allspectra exceeding the best single modality by roughly +0.3%.
- Each eval run samples exactly 10,000 molecules per test spectrum (config.eval.num_samples 10000) and loads checkpoint 40 (config.eval.ckpts '40').
- Generated molecules and metrics land in exp/{exp_name}/eval/ for each of the four workdirs (exp/ir, exp/raman, exp/uv, exp/allspectra[_pretrained_specformer]).
- The lowest MCES value among all conditions is attained by the joint-modality run (reported 1.780).

## Limitations

- Validation is primarily computational; the article notes 'our validation to date has been primarily computational. Therefore, extensive experimental studies will be an important next step'.
- Only three spectral modalities are covered; NMR and mass spectrometry are explicitly not supported.
- Results are tied to the QM9S dataset (QM9 augmented with simulated spectra) and small QM9-scale molecules; generalizability to larger, real-spectra settings is untested.
- Multi-modal gains over the best single modality are modest (~+0.3% top-1), so the ablation conclusion may be sensitive to sampling variance.

## Evidence
