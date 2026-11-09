---
name: spectrum-conditioned-generative-sampling
description: Use when you have trained DMT checkpoints and multi-modal spectra (IR/Raman/UV-Vis)
  and need to generate de novo 3D molecular structures via spectra-conditioned diffusion
  sampling, then score exact-match accuracy.
when_to_use_negative:
- You need structure elucidation from NMR or mass spectrometry data — DiffSpectra
  currently focuses only on UV-Vis, IR, and Raman spectra
- You only need unconditional molecular generation benchmarks (e.g. comparing against
  CDGS or JODO) rather than spectra-conditioned elucidation
- You have no trained checkpoints and only want to pre-train the spectral encoder
  — use the SpecFormer pre-training pipeline (MPR + InfoNCE) instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: spectra-conditioned diffusion framework whose reverse denoising process generates
    molecular structures from IR/Raman/UV-Vis input
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network that predicts the data at each reverse-diffusion
    step over node, edge, and coordinate streams
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder providing conditioning embeddings; optionally
    loaded from a pretrained checkpoint to improve accuracy
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:f1ef8544dc883f43324fa8c43c65eff6e5c4b85b4fee2d9e93fe0af225ec938b
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectrum-conditioned-generative-sampling

## Summary

Runs DiffSpectra's reverse-time diffusion denoising (t from 1 to 0) to generate candidate 2D topologies and 3D coordinates conditioned on multi-modal spectra (IR, Raman, UV-Vis) encoded by SpecFormer, then computes top-1/top-10 accuracy and MCES metrics on the sampled molecules.

## When to use

Use when you have trained DMT checkpoints and multi-modal spectra (IR/Raman/UV-Vis) and need to generate de novo 3D molecular structures via spectra-conditioned diffusion sampling, then score exact-match accuracy.

## When NOT to use

- You need structure elucidation from NMR or mass spectrometry data — DiffSpectra currently focuses only on UV-Vis, IR, and Raman spectra
- You only need unconditional molecular generation benchmarks (e.g. comparing against CDGS or JODO) rather than spectra-conditioned elucidation
- You have no trained checkpoints and only want to pre-train the spectral encoder — use the SpecFormer pre-training pipeline (MPR + InfoNCE) instead

## Inputs

- QM9S dataset (QM9 molecules augmented with simulated spectra) placed at the configured data.root path
- Trained DMT model checkpoints (e.g. checkpoint 40) from huggingface.co/AzureLeon1/DiffSectra placed in the exp/ workdir
- Optional pretrained SpecFormer checkpoint (exp/pretrained_specformer.ckpt)
- Config files configs/diffspectra_qm9s.py and configs/base_qm9.py

## Outputs

- Generated molecules (2D topology + 3D coordinates) saved with --config.eval.save_mols true
- Top-1 / top-10 exact-match accuracy and similarity metrics in exp/{exp_name}/eval/
- RMSD values from atom-mapped 3D structural evaluation
- Results tables (e.g. results_table.csv) comparing accuracy across configuration ablations

## How to apply

Configure the spectra-conditioned generation in configs/diffspectra_qm9s.py with --config.data.spectra_version (allspectra, ir, raman, or uv) and set --config.model.name DMT; point --config.data.root at the QM9S dataset and --workdir at the directory holding the trained checkpoints. Run the reverse denoising process from a standard Gaussian sample, proceeding backward from t=1 to t=0, where the DMT denoiser (parameterized to preserve permutation and SE(3) equivariance) predicts data at each step guided by SpecFormer spectral embeddings; use --mode eval with --config.eval.ckpts (e.g. '40') and --config.eval.num_samples (e.g. 10000) and --config.eval.save_mols true. Metrics (accuracy and similarity) are computed automatically during generation, or manually via eval_sampled_mols/base_metrics.py; 3D structural evaluation (RMSD against ground truth using Hungarian atom mapping) uses eval_sampled_mols/rmsd.py. Results land in exp/{exp_name}/eval/. For ablation conditions (e.g. comparing SE(3)-equivariant architecture alone vs. augmentation-based equivariance), toggle the augmentation setting in the DMT configuration and rerun the identical eval command, then compare Acc@1 and MCES values.

## Related tools

- **DiffSpectra** (spectra-conditioned diffusion framework whose reverse denoising process generates molecular structures from IR/Raman/UV-Vis input) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network that predicts the data at each reverse-diffusion step over node, edge, and coordinate streams) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder providing conditioning embeddings; optionally loaded from a pretrained checkpoint to improve accuracy) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- With all spectra and pretrained SpecFormer, sampled generation should approach the reported 40.76% top-1 and 99.49% top-10 accuracy on QM9S
- Ablation check: removing augmentation from an SE(3)-equivariant DMT configuration should degrade Acc@1 to ~20.25% and MCES to ~3.6036, while the augmented variant reaches ~31.24% Acc@1 / 1.7795 MCES
- Generated molecules and metrics must appear under exp/{exp_name}/eval/ after the eval run
- RMSD evaluation requires valid Hungarian-algorithm atom mapping against ground-truth 3D structures
- 3D structural metrics (e.g. via eval_sampled_mols/rmsd.py) should confirm generated coordinates are chemically plausible, not just topologically correct

## Limitations

- Framework limited to three spectral types (UV-Vis, IR, Raman); applicability to NMR or mass spectrometry remains unexplored
- Validation is primarily computational; no experimental/laboratory validation has been performed
- The paper reports no software versions, model hyperparameters, or deep-learning library details; environment must be installed via env.sh
- Accuracy depends on spectral modality coverage — single-spectrum (IR-only, Raman-only, UV-only) configurations yield lower accuracy than allspectra conditioning

## Evidence

- [methods] reverse denoising: [evidence span withheld — anchor status: not_found]
