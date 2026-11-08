---
name: exact-structure-matching-metrics
description: Use when you have generated molecular structures (e.g., from a diffusion
  or autoregressive model) plus ground-truth structures and need exact-match top-k
  accuracy or MCES similarity scores.
when_to_use_negative:
- You need 3D conformational fidelity assessment rather than 2D topological identity
  — use the RMSD/Hungarian-mapping evaluation (rmsd.py) instead
- Your generated outputs are not full molecular structures (e.g., spectra-only predictions
  or property values) — exact-match comparison is undefined
- You are evaluating unconditional generative quality metrics (validity, novelty,
  uniqueness) rather than recovery of a known target structure
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: Generates candidate molecules via spectrum-conditioned diffusion; metrics
    computed automatically during its eval mode
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: RDKit
  role: Canonicalizes generated structures and supports exact-match and MCES comparison
    against ground truth
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising backbone whose generated molecules feed the matching
    metrics
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:97e6e51b3f35ca581a660f6e4f80fb09c3d37f4936b1b74b37cba3a260b98b58
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# Exact-structure matching metrics

## Summary

Evaluates spectra-conditioned molecular generation by comparing canonical structures of generated molecules against ground truth to compute exact-match top-1/top-10 accuracy and MCES distance, as done for DiffSpectra on QM9S.

## When to use

Use when you have generated molecular structures (e.g., from a diffusion or autoregressive model) plus ground-truth structures and need exact-match top-k accuracy or MCES similarity scores.

## When NOT to use

- You need 3D conformational fidelity assessment rather than 2D topological identity — use the RMSD/Hungarian-mapping evaluation (rmsd.py) instead
- Your generated outputs are not full molecular structures (e.g., spectra-only predictions or property values) — exact-match comparison is undefined
- You are evaluating unconditional generative quality metrics (validity, novelty, uniqueness) rather than recovery of a known target structure

## Inputs

- Generated molecules saved from eval mode (exp/{exp_name}/eval/ output of DiffSpectra generation)
- Ground-truth molecular structures from the QM9S test set
- Trained DiffSpectra (DMT) checkpoint from huggingface.co/AzureLeon1/DiffSpectra

## Outputs

- Exact-match top-1 accuracy (e.g., 40.76% for allspectra, 35.6% reported in ablation table)
- Exact-match top-10 accuracy (e.g., 99.49%)
- MCES distance per prediction (lowest 1.780 for all-modal conditioning)
- Saved generated molecules and metrics under exp/{exp_name}/eval/

## How to apply

Generate molecules conditioned on spectra (e.g., 10,000 samples per spectrum via reverse-time denoising from t=1 to t=0 with the DMT backbone), then canonicalize each generated structure and compare against the ground truth with RDKit to compute exact-match top-1 and top-10 accuracy. Compute Maximum Common Edge Subgraph (MCES) distance as a graded similarity measure for near-miss structures. Use the repository's eval_sampled_mols/base_metrics.py, which runs automatically alongside generation when metrics are requested, or invoke it manually on saved molecules. Aggregate per-modality results (ir, raman, uv, allspectra) into an ablation table, validating against reference numbers such as 40.76% top-1 / 99.49% top-10 for the multi-modal setting. For 3D correctness, separately run eval_sampled_mols/rmsd.py with Hungarian atom mapping against ground-truth coordinates.

## Related tools

- **DiffSpectra** (Generates candidate molecules via spectrum-conditioned diffusion; metrics computed automatically during its eval mode) — https://github.com/AzureLeon1/DiffSpectra
- **RDKit** (Canonicalizes generated structures and supports exact-match and MCES comparison against ground truth)
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising backbone whose generated molecules feed the matching metrics) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
python eval_sampled_mols/base_metrics.py
```

## Evaluation signals

- Full multi-modal ('allspectra') setting reproduces the reported headline accuracy of 40.76% top-1 and 99.49% top-10
- Top-1 accuracy of the allspectra setting exceeds the best single-spectrum setting (Raman, 30.3%) by approximately +0.3%
- Single-modality ordering holds: Raman (30.3%) > IR (25.9%) > UV-Vis (0.3%) top-1 accuracy
- Metrics files appear at exp/{exp_name}/eval/ after running base_metrics.py
- All generated structures pass RDKit canonicalization without sanitization errors

## Limitations

- Exact-match top-1 is a stringent all-or-nothing metric; MCES is needed to capture near-miss structural similarity
- Metrics are 2D topological; separate RMSD evaluation with Hungarian atom mapping is required to verify 3D geometric correctness
- Validation is primarily computational — no experimental laboratory validation of elucidated structures
- Framework covers only IR, Raman, and UV-Vis; results do not transfer to NMR or mass spectrometry modalities

## Evidence

- [results] compare the canonical structure against the ground truth to compute exact-match top-1 and top-10 accuracy using the repository's base_metrics.py: [evidence span withheld — anchor status: not_found]
