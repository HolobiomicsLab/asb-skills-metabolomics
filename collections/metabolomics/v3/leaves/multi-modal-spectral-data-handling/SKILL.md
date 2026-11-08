---
name: multi-modal-spectral-data-handling
description: Use when a dataset provides multiple spectrum types per molecule (IR,
  Raman, UV-Vis) and the goal is to condition structure elucidation or generation
  on all modalities jointly rather than a single spectrum type.
when_to_use_negative:
- The target data contains spectral modalities outside the supported set — DiffSpectra
  currently handles only UV-Vis, IR, and Raman, and applicability to NMR or mass spectrometry
  remains unexplored.
- Only one spectrum type is available and a joint multi-modal run is not needed —
  in that case use spectra_version ir, raman, or uv instead of allspectra.
- The task is spectrum simulation or spectral pre-processing (e.g. smoothing) rather
  than spectra-conditioned molecular structure generation.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: Generative framework that formulates structure elucidation as a conditional
    diffusion process conditioned on multi-modal spectral data
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder that patches and encodes IR, Raman, and
    UV-Vis spectra jointly; pre-trained with masked patches reconstruction and contrastive
    alignment against 3D structures
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network guided by the multi-modal SpecFormer embeddings
    during reverse diffusion
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:36d1c652fa6d49414fb8118764920e210dfdd0a2d8779e378b32cbe6dca7d214
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - Diffusion Molecule Transformer (DMT)
  - Hungarian algorithm
  - SpecFormer
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multi-modal-spectral-data-handling

## Summary

Handling and joint integration of multi-modal molecular spectra (IR, Raman, UV-Vis) so that all three spectral modalities are supplied together as conditioning input for spectra-conditioned molecular generation and evaluation with DiffSpectra.

## When to use

Use when a dataset provides multiple spectrum types per molecule (IR, Raman, UV-Vis) and the goal is to condition structure elucidation or generation on all modalities jointly rather than a single spectrum type.

## When NOT to use

- The target data contains spectral modalities outside the supported set — DiffSpectra currently handles only UV-Vis, IR, and Raman, and applicability to NMR or mass spectrometry remains unexplored.
- Only one spectrum type is available and a joint multi-modal run is not needed — in that case use spectra_version ir, raman, or uv instead of allspectra.
- The task is spectrum simulation or spectral pre-processing (e.g. smoothing) rather than spectra-conditioned molecular structure generation.

## Inputs

- QM9S dataset (QM9 molecules augmented with simulated IR, Raman, and UV-Vis spectra) downloaded from huggingface.co/datasets/AzureLeon1/DiffSpectra
- Pretrained SpecFormer checkpoint (e.g. exp/pretrained_specformer.ckpt from huggingface.co/AzureLeon1/DiffSpectra)
- Configuration file configs/diffspectra_qm9s.py with data.root pointing to the dataset directory
- spectra_version setting: allspectra (or ir / raman / uv for single-modality baselines)

## Outputs

- Spectra-conditioned generated molecular structures (2D topology plus 3D coordinates) written to exp/{exp_name}/eval/
- Accuracy and similarity metrics (Acc@1, Acc@10, molecular stability, uniqueness, novelty, FCD) from eval_sampled_mols/base_metrics.py
- 3D RMSD metrics from eval_sampled_mols/rmsd.py (Hungarian atom mapping)

## How to apply

Prepare the QM9S spectra files (QM9 augmented with simulated spectra, hosted on huggingface.co/datasets/AzureLeon1/DiffSpectra) and set data.root in configs/diffspectra_qm9s.py to that directory. Select the joint-modality conditioning by passing --config.data.spectra_version allspectra so IR, Raman, and UV-Vis are supplied together; for controlled comparisons, rerun the same eval command with spectra_version set to ir, raman, or uv individually. Each spectrum type is segmented into patches and encoded by SpecFormer, a Transformer-based encoder pre-trained with masked patches reconstruction (MPR) and contrastive alignment (InfoNCE) against 3D structures, so use the pretrained SpecFormer checkpoint (e.g. exp/pretrained_specformer.ckpt) via --config.model.pretrained_specformer_path for the best accuracy. The resulting multi-modal embeddings condition the DMT denoiser (SE(3)-equivariant) during reverse diffusion from t=1 to t=0. Correct application is confirmed when the joint-modality run recovers structures markedly better than single-modality runs and matches the reported allspectra metrics.

## Related tools

- **DiffSpectra** (Generative framework that formulates structure elucidation as a conditional diffusion process conditioned on multi-modal spectral data) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder that patches and encodes IR, Raman, and UV-Vis spectra jointly; pre-trained with masked patches reconstruction and contrastive alignment against 3D structures) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network guided by the multi-modal SpecFormer embeddings during reverse diffusion) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.data.spectra_version allspectra
```

## Evaluation signals

- The eval directory exp/{exp_name}/eval/ contains the generated molecule files and metric outputs after the run.
- The allspectra run with pretrained SpecFormer reproduces approximately Acc@1 = 40.76%, Acc@10 = 99.49%, molecular stability ~98.2%, and FCD ~0.088.
- Accuracy and similarity metrics computed automatically with generation (or manually via eval_sampled_mols/base_metrics.py) are present in results_table.csv.
- Joint multi-modal conditioning (allspectra) outperforms single-modality runs (ir, raman, uv) run with otherwise identical eval settings.
- The SpecFormer path passed via --config.model.pretrained_specformer_path resolves to a valid checkpoint file before training/eval.

## Limitations

- DiffSpectra currently focuses on three spectral types (UV-Vis, IR, Raman); applicability to NMR or mass spectrometry remains to be explored.
- Validation is primarily computational; no experimental/laboratory validation has been performed.
- No software versions, model hyperparameters, or deep-learning library details are stated; only env.sh is provided for environment installation.
- Generalizability depends on the diversity and scale of available spectral datasets.

## Evidence

- [methods] contrastive alignment pre-training: [evidence span withheld — anchor status: not_found]
