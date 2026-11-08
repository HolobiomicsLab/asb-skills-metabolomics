---
name: fingerprint-similarity-metrics
description: Use when scoring de novo generated molecules against known ground-truth
  structures on a spectral elucidation benchmark, or when quantifying how much an
  ablation (e.g., 2D-only vs joint 2D&3D diffusion) degrades structural fidelity beyond
  exact-match accuracy.
when_to_use_negative:
- When there is no ground-truth structure to compare against (pure unconditional generation
  quality assessment needs validity/novelty metrics instead)
- When evaluating elucidation on spectral modalities outside IR/Raman/UV-Vis (e.g.,
  NMR or MS), which DiffSpectra does not cover
- When exact atom-level correspondence is the only question — RMSD with Hungarian
  mapping is the appropriate metric, not fingerprint Tanimoto
edam_operation: http://edamontology.org/operation_3352
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: Generates candidate structures via spectra-conditioned joint 2D/3D diffusion
    and auto-computes accuracy/similarity metrics during eval mode
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: eval_sampled_mols/base_metrics.py
  role: Standalone script for computing exact-structure accuracy and fingerprint-based
    similarity metrics from saved generated molecules
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: eval_sampled_mols/rmsd.py
  role: Computes RMSD after Hungarian atom mapping between generated and ground-truth
    3D coordinates
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:91ea9309193a85fc2872576bce4b72ad91ab4f521c29459e9fcd85b942fa071c
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - SpecFormer
  - eval_sampled_mols/base_metrics.py
  - eval_sampled_mols/rmsd.py
  edam_operation: http://edamontology.org/operation_3352
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# fingerprint-similarity-metrics

## Summary

Evaluate spectra-conditioned molecular generation by comparing generated SMILES against ground-truth structures using fingerprint-based similarities (Tanimoto over Morgan/MA fingerprints), functional-group similarity (FGSim), MCES distance, and exact top-1/top-10 accuracy, as done in DiffSpectra's ablation and benchmark tables.

## When to use

Use when scoring de novo generated molecules against known ground-truth structures on a spectral elucidation benchmark, or when quantifying how much an ablation (e.g., 2D-only vs joint 2D&3D diffusion) degrades structural fidelity beyond exact-match accuracy.

## When NOT to use

- When there is no ground-truth structure to compare against (pure unconditional generation quality assessment needs validity/novelty metrics instead)
- When evaluating elucidation on spectral modalities outside IR/Raman/UV-Vis (e.g., NMR or MS), which DiffSpectra does not cover
- When exact atom-level correspondence is the only question — RMSD with Hungarian mapping is the appropriate metric, not fingerprint Tanimoto

## Inputs

- Generated molecules from DiffSpectra sampling (SMILES with optional 3D coordinates, saved under exp/{exp_name}/eval/)
- Ground-truth structures from the QM9S test set (HuggingFace: huggingface.co/datasets/AzureLeon1/DiffSpectra)
- Evaluation config: config.eval.num_samples=10000, config.eval.ckpts=40, config.data.spectra_version

## Outputs

- Per-variant metrics: top-1 and top-10 exact-structure accuracy, TaniSim_MG, TaniSim_MA, FGSim, MCES distance
- RMSD statistics from Hungarian-mapped coordinate alignment (joint 2D&3D variant only)
- CSV table with one row per variant (2D-only vs joint 2D&3D) and columns for accuracy, similarity, and RMSD

## How to apply

Run DiffSpectra sampling (e.g., 10,000 samples, eval.ckpts=40) on QM9S test spectra in the chosen spectra mode ('allspectra', 'ir', 'raman', or 'uv'), then compare each generated molecule's SMILES to the ground-truth structure. Compute top-1 and top-10 exact-match accuracy plus fingerprint similarities — reported as TaniSim_MG (Morgan/EGAS fingerprint Tanimoto, e.g., 0.5609 for 2D-only vs 0.7875 for joint 2D&3D), TaniSim_MA (0.8350→0.9262), and FGSim functional-group similarity (0.9034→0.9652) — along with MCES distance. Metrics are computed automatically during generation, or manually via eval_sampled_mols/base_metrics.py. For joint 2D&3D variants, additionally align generated 3D coordinates to ground truth via Hungarian atom mapping and record RMSD using eval_sampled_mols/rmsd.py. Aggregate per-variant rows into a CSV to expose which similarity metrics show the ablation gap.

## Related tools

- **DiffSpectra** (Generates candidate structures via spectra-conditioned joint 2D/3D diffusion and auto-computes accuracy/similarity metrics during eval mode) — https://github.com/AzureLeon1/DiffSpectra
- **eval_sampled_mols/base_metrics.py** (Standalone script for computing exact-structure accuracy and fingerprint-based similarity metrics from saved generated molecules) — https://github.com/AzureLeon1/DiffSpectra
- **eval_sampled_mols/rmsd.py** (Computes RMSD after Hungarian atom mapping between generated and ground-truth 3D coordinates) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
python eval_sampled_mols/base_metrics.py
```

## Evaluation signals

- Joint 2D&3D variant must reproduce reported accuracies within diffusion-sampling tolerance: top-1 ≈ 40.76% and top-10 ≈ 99.49%
- 2D-only ablation must show the reported drop to top-1 ≈ 9.49%, confirming 3D geometric modeling benefits
- Similarity metrics should show consistent improvements in the joint variant: TaniSim_MG 0.5609→0.7875, TaniSim_MA 0.8350→0.9262, FGSim 0.9034→0.9652
- MCES distance should be reduced for the joint 2D&3D variant relative to 2D-only
- Metrics files present at exp/{exp_name}/eval/ for both variants before aggregation

## Limitations

- Run-to-run variance from diffusion sampling means exact metric reproduction requires tolerance bands rather than exact equality
- Validation is primarily computational; no experimental/laboratory validation of elucidated structures has been performed
- Fingerprint similarities are approximate structural measures and cannot substitute for exact-match accuracy or RMSD when atom-level correspondence matters

## Evidence

- [other] exact top-1 elucidation accuracy rises from 9.49% to 40.76%, with reduced MCES and consistent improvements in fingerprint, fragment, and functional-group similarities: [evidence span withheld — anchor status: not_found]
