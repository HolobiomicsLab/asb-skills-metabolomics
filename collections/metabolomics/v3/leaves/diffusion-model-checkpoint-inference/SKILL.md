---
name: diffusion-model-checkpoint-inference
description: Use when trained DiffSpectra/DMT checkpoints and multi-modal spectra
  (IR, Raman, UV-Vis) from QM9S are available and the goal is spectra-conditioned
  molecular generation with reproducible top-k accuracy metrics.
when_to_use_negative:
- Input spectra are NMR or mass spectrometry data — the framework covers only UV-Vis,
  IR, and Raman modalities, and its applicability to other modalities remains unexplored.
- The goal is training or pre-training SpecFormer (masked patches reconstruction /
  contrastive alignment) rather than inference from an existing checkpoint.
- The task requires experimental/laboratory validation of structures — the paper's
  validation is primarily computational.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3332
tools:
- name: DiffSpectra
  role: Spectra-conditioned diffusion framework invoked in eval mode to generate molecules
    from Gaussian noise; main.py drives the eval pipeline and metrics.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network that jointly generates 2D topology and
    3D coordinates during the reverse denoising process at checkpoint 40.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Pre-trained spectral encoder providing spectral embeddings that condition
    the DMT denoiser on IR, Raman, and UV-Vis modalities.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Hungarian algorithm
  role: Atom-to-atom mapping method used by eval_sampled_mols/rmsd.py to align generated
    3D structures with ground truth for RMSD computation.
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_001
  source_content_hash: sha256:4841d082efd833eeb17d39004feca7786c6931962b2161d83771c4c31cdf0f0a
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# diffusion-model-checkpoint-inference

## Summary

Run a trained DiffSpectra checkpoint (DMT denoiser conditioned on SpecFormer spectral embeddings) in eval mode to generate molecules from Gaussian noise via the reverse denoising process, then score generated structures against ground truth for exact-structure accuracy and quality metrics.

## When to use

Use when trained DiffSpectra/DMT checkpoints and multi-modal spectra (IR, Raman, UV-Vis) from QM9S are available and the goal is spectra-conditioned molecular generation with reproducible top-k accuracy metrics.

## When NOT to use

- Input spectra are NMR or mass spectrometry data — the framework covers only UV-Vis, IR, and Raman modalities, and its applicability to other modalities remains unexplored.
- The goal is training or pre-training SpecFormer (masked patches reconstruction / contrastive alignment) rather than inference from an existing checkpoint.
- The task requires experimental/laboratory validation of structures — the paper's validation is primarily computational.

## Inputs

- QM9S dataset (QM9 augmented with simulated IR, Raman, and UV-Vis spectra) from huggingface.co/datasets/AzureLeon1/DiffSpectra
- Pretrained SpecFormer checkpoint (e.g. exp/pretrained_specformer.ckpt)
- Trained DiffSpectra/DMT model checkpoints (e.g. allspectra_pretrained_specformer, epoch 40) from huggingface.co/AzureLeon1/DiffSpectra
- Config file configs/diffspectra_qm9s.py with data.root pointing to the dataset path
- Ground-truth 2D SMILES and 3D structures of the QM9S test set for metric computation

## Outputs

- Generated molecule files and evaluation metrics in exp/{exp_name}/eval/
- Top-1 (Acc@1) and top-10 (Acc@10) exact-structure accuracy (expected ~40.76% and ~99.49%)
- FCD (~0.088), molecular stability (~98.2%), uniqueness, novelty, and atom/molecular stability metrics
- RMSD values from Hungarian-algorithm atom mapping against ground-truth 3D structures
- Consolidated results_table.csv in exp/{exp_name}/eval/

## How to apply

Place the QM9S dataset in the configured data.root directory and load the pretrained SpecFormer and DiffSpectra checkpoints (e.g. allspectra_pretrained_specformer, checkpoint 40) into exp/. Run main.py with --mode eval and configs/diffspectra_qm9s.py, setting --config.eval.ckpts '40', --config.eval.num_samples 10000, --config.eval.save_mols true, and --config.data.spectra_version allspectra so the reverse-time denoising process (t from 1 to 0) transforms a standard Gaussian sample into molecules conditioned on the spectral embeddings. Accuracy and similarity metrics are computed automatically during generation, or manually via eval_sampled_mols/base_metrics.py matching generated molecules to ground truth by SMILES identity; 3D quality is computed by eval_sampled_mols/rmsd.py, which performs atom mapping against ground-truth 3D structures using the Hungarian algorithm and reports RMSD. Ablation variants (random vs pretrained SpecFormer, augmentation on/off, per-spectra-version IR/Raman/UV runs) reuse the same eval pipeline with the corresponding checkpoint and spectra_version flags.

## Related tools

- **DiffSpectra** (Spectra-conditioned diffusion framework invoked in eval mode to generate molecules from Gaussian noise; main.py drives the eval pipeline and metrics.) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network that jointly generates 2D topology and 3D coordinates during the reverse denoising process at checkpoint 40.) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Pre-trained spectral encoder providing spectral embeddings that condition the DMT denoiser on IR, Raman, and UV-Vis modalities.) — https://github.com/AzureLeon1/DiffSpectra
- **Hungarian algorithm** (Atom-to-atom mapping method used by eval_sampled_mols/rmsd.py to align generated 3D structures with ground truth for RMSD computation.)

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Reproduced Acc@1 is approximately 40.76% and Acc@10 approximately 99.49% on the QM9S test set (num_samples=10000, checkpoint 40).
- FCD is approximately 0.088 and molecular stability approximately 98.2%, matching the paper's reported values.
- The exp/{exp_name}/eval/ directory contains both the generated molecule files and the metric outputs (results_table.csv).
- Molecules match ground truth by exact SMILES identity for top-k accuracy counting.
- Ablation sanity check: with-augmentation Acc@1 (35.64% baseline config) exceeds the augmentation-removed variant (20.25%), and pretrained SpecFormer (+40.76%) exceeds random-init SpecFormer (35.64%).

## Limitations

- Validation is primarily computational; no experimental/laboratory validation has been performed.
- Only three spectral types are supported (UV-Vis, IR, Raman); NMR and mass spectrometry remain to be explored.
- Evaluation relies on simulated spectra over QM9-derived molecules (QM9S); no software versions or dependency details are stated beyond env.sh.
- Metrics depend on sampling scale and checkpoint choice (num_samples=10000, checkpoint '40' in the reported configuration).

## Evidence

- [abstract] Extensive experiments demonstrate that DiffSpectra accurately elucidates molecular structures, achieving 40.76% top-1 and 99.49% top-10 accuracy.: "Extensive experiments demonstrate that DiffSpectra accurately elucidates molecular structures, achieving 40.76% top-1 and 99.49% top-10 accuracy."
