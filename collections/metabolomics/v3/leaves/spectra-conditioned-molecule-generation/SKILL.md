---
name: spectra-conditioned-molecule-generation
description: Use when you must recover an unknown molecule's exact structure from
  simulated or measured multi-modal spectra (IR, Raman, UV-Vis), especially when 3D
  conformation matters and single-spectrum methods are insufficient.
when_to_use_negative:
- Spectra are limited to modalities outside UV-Vis, IR, and Raman — the framework's
  applicability to NMR or mass spectrometry is explicitly unexplored
- You only need a single-spectrum (IR-only, Raman-only, or UV-Vis-only) elucidation
  result and the multi-modal allspectra conditioning is unnecessary — although the
  same pipeline supports spectra_version ir/raman/uv
- The target molecules lie far outside the QM9-like chemical space; the paper's validation
  is purely computational on QM9S (QM9 augmented with simulated spectra) with no experimental
  validation
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: Overall spectra-conditioned diffusion framework whose eval mode generates
    molecules from Gaussian noise guided by spectral embeddings
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network that jointly generates 2D topology and
    3D coordinates conditioned on SpecFormer embeddings
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder (pre-trained via masked patches reconstruction
    and contrastive alignment) that conditions the DMT denoiser; its pretrained checkpoint
    materially improves accuracy
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Hungarian algorithm
  role: Used in eval_sampled_mols/rmsd.py to atom-map generated 3D structures to ground
    truth before computing RMSD
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:21ed8ccd3e865cb5a410e07ef12f6151a84ede2c5079cf681c928f1731fa54da
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectra-conditioned-molecule-generation

## Summary

Generate complete 3D molecular structures from multi-modal spectra (IR, Raman, UV-Vis) by running a SE(3)-equivariant diffusion denoiser (DMT) conditioned on SpecFormer spectral embeddings, as in DiffSpectra. It replaces retrieval-based or SMILES-only elucidation with joint 2D topology and 3D geometry generation.

## When to use

Use when you must recover an unknown molecule's exact structure from simulated or measured multi-modal spectra (IR, Raman, UV-Vis), especially when 3D conformation matters and single-spectrum methods are insufficient.

## When NOT to use

- Spectra are limited to modalities outside UV-Vis, IR, and Raman — the framework's applicability to NMR or mass spectrometry is explicitly unexplored
- You only need a single-spectrum (IR-only, Raman-only, or UV-Vis-only) elucidation result and the multi-modal allspectra conditioning is unnecessary — although the same pipeline supports spectra_version ir/raman/uv
- The target molecules lie far outside the QM9-like chemical space; the paper's validation is purely computational on QM9S (QM9 augmented with simulated spectra) with no experimental validation

## Inputs

- Multi-modal spectral data (IR, Raman, UV-Vis) for each query molecule, from the QM9S dataset (huggingface.co/datasets/AzureLeon1/DiffSpectra) placed in data.root
- Pretrained SpecFormer checkpoint (e.g. exp/pretrained_specformer.ckpt)
- Trained DiffSpectra/DMT model checkpoints (e.g. allspectra_pretrained_specformer, epoch 40) from huggingface.co/AzureLeon1/DiffSpectra
- Ground-truth 2D/3D reference structures for the test set (needed for Acc@1/Acc@10 and RMSD evaluation)

## Outputs

- Generated 3D molecular structures (saved molecule files) per spectrum in exp/{exp_name}/eval/
- Exact-structure accuracy metrics: Acc@1 (~40.76%), Acc@10 (~99.49%), plus molecular stability (~98.2%), uniqueness, novelty, and FCD (~0.088)
- RMSD values against ground-truth 3D structures after Hungarian-algorithm atom mapping
- results_table.csv consolidating all computed metrics in exp/{exp_name}/eval/

## How to apply

Set up the DiffSpectra environment via env.sh, download the QM9S dataset into data.root, and place the pretrained SpecFormer checkpoint (e.g. pretrained_specformer.ckpt) and DMT checkpoints (e.g. allspectra_pretrained_specformer epoch 40) in exp/. Run main.py in eval mode with configs/diffspectra_qm9s.py, --config.data.spectra_version allspectra, --config.eval.ckpts '40', --config.eval.num_samples 10000, and --config.eval.save_mols true so the DMT denoises molecules from Gaussian noise conditioned on SpecFormer embeddings of the spectra patches. Metrics for accuracy and similarity are computed automatically during generation or manually with eval_sampled_mols/base_metrics.py (Acc@1, Acc@10, stability, uniqueness, novelty, FCD); 3D quality uses eval_sampled_mols/rmsd.py, which maps atoms to ground truth with the Hungarian algorithm before computing RMSD. Consolidate generated molecule files and metrics from exp/{exp_name}/eval/ into a results table and compare against the reported 40.76% Acc@1 and 99.49% Acc@10.

## Related tools

- **DiffSpectra** (Overall spectra-conditioned diffusion framework whose eval mode generates molecules from Gaussian noise guided by spectral embeddings) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network that jointly generates 2D topology and 3D coordinates conditioned on SpecFormer embeddings) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder (pre-trained via masked patches reconstruction and contrastive alignment) that conditions the DMT denoiser; its pretrained checkpoint materially improves accuracy) — https://github.com/AzureLeon1/DiffSpectra
- **Hungarian algorithm** (Used in eval_sampled_mols/rmsd.py to atom-map generated 3D structures to ground truth before computing RMSD) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Reproduced Acc@1 is approximately 40.76% and Acc@10 approximately 99.49% on the QM9S test set
- Molecular stability is approximately 98.2% and FCD approximately 0.088, matching the paper's reported values
- exp/{exp_name}/eval/ contains both the generated molecule files (--config.eval.save_mols true) and the metric outputs
- RMSD computation succeeds, implying valid Hungarian-algorithm atom mapping between generated and ground-truth 3D structures
- All consolidated metrics land in results_table.csv under exp/{exp_name}/eval/

## Limitations

- Only three spectral types (UV-Vis, IR, Raman) are supported; NMR and mass spectrometry remain unexplored
- Validation is primarily computational on QM9S simulated spectra; no experimental/laboratory validation has been performed
- No software versions or library-level implementation details (e.g. which diffusion/deep-learning library) are stated, so reproduction depends on env.sh pinning the environment
- Full multi-sample generation uses num_samples 10000, which can be computationally expensive; single-attempt top-1 accuracy (40.76%) is far lower than top-10 (99.49%), so sampling budget strongly affects hit rate

## Evidence
