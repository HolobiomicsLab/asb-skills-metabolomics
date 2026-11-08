---
name: molecular-topology-accuracy-evaluation
description: Use when you have generated candidate molecules (2D topology from a diffusion
  or generative model) paired with known reference structures and need to quantify
  exact-match and similarity-based elucidation accuracy.
when_to_use_negative:
- When the goal is 3D conformational fidelity rather than 2D topology — use RMSD-based
  evaluation (eval_sampled_mols/rmsd.py) instead.
- When evaluating unconditional generation quality (validity, uniqueness, novelty)
  rather than recovering a specific target molecule from spectra.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0091
tools:
- name: DiffSpectra
  role: Generates candidate molecular structures conditioned on spectra and computes
    accuracy/similarity metrics during evaluation
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: eval_sampled_mols/base_metrics.py
  role: Manually recomputes accuracy and similarity metrics on previously generated
    molecules
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:e6b4a51cfe1e8315bd8d2644208bba263c950e656dcfc2bc5da77417d9ee31d6
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# molecular-topology-accuracy-evaluation

## Summary

Evaluates whether spectra-conditioned diffusion samples (e.g., DiffSpectra on QM9S) reproduce the ground-truth molecule's atom types and bond types, reporting top-1/top-10 accuracy plus similarity metrics such as MCES.

## When to use

Use when you have generated candidate molecules (2D topology from a diffusion or generative model) paired with known reference structures and need to quantify exact-match and similarity-based elucidation accuracy.

## When NOT to use

- When the goal is 3D conformational fidelity rather than 2D topology — use RMSD-based evaluation (eval_sampled_mols/rmsd.py) instead.
- When evaluating unconditional generation quality (validity, uniqueness, novelty) rather than recovering a specific target molecule from spectra.

## Inputs

- Generated molecular structures (atom types, bond types, and optionally coordinates) from the diffusion sampler
- Ground-truth reference molecules from QM9S (SMILES or molecular graphs with atom/bond types)

## Outputs

- Top-1 and top-10 accuracy percentages
- Similarity metrics such as MCES between generated and reference molecules
- Per-sample generation logs and saved molecule files under exp/{exp_name}/eval/

## How to apply

Run generation in eval mode (e.g., `--mode eval` with `--config.eval.num_samples 10000` and `--config.eval.save_mols true`) so top-1/top-10 accuracy and similarity are computed automatically during sampling; alternatively compute them post hoc via `eval_sampled_mols/base_metrics.py`. Compare each generated molecule's atom and bond types against the ground-truth QM9S reference structure, recording how often the exact structure appears as the highest- or among the top-k samples. Report match rates at the sample level (e.g., 40.76% top-1 / 99.49% top-10 for all-spectra with pretrained SpecFormer) and complement them with similarity measures such as maximum common edge substructure (MCES). Ablations sweep conditions like sampling temperature (τ = 0.0–2.0) using the same metric suite, so keep the metric computation fixed across settings for fair comparison.

## Related tools

- **DiffSpectra** (Generates candidate molecular structures conditioned on spectra and computes accuracy/similarity metrics during evaluation) — https://github.com/AzureLeon1/DiffSpectra
- **eval_sampled_mols/base_metrics.py** (Manually recomputes accuracy and similarity metrics on previously generated molecules) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Top-1 accuracy for the full setup should be on the order of ~40.76% (all spectra, pretrained SpecFormer); a value near 14.65% indicates the τ=0.0 ablation setting.
- Top-10 accuracy should be very high (~99.49%) when multi-modal spectra and pretrained SpecFormer are used.
- Automated metric computation must report both accuracy and similarity (MCES) values alongside the saved molecules in exp/{exp_name}/eval/.
- When sweeping sampling temperature, per-τ accuracy values should trace an inverted-U trend with the peak at τ=1.0 (τ=0.0 → 14.65%, τ=1.2 → 39.59%).
- Molecule files and metric logs must be present in the eval directory after a successful run.

## Limitations

- Accuracy is computed only against the single ground-truth QM9S structure, so valid alternative isomers count as misses.
- Eval is tied to the QM9S spectral simulation pipeline; applicability to experimental spectra (or other modalities like NMR/MS) is untested.
- Computational evaluation only — no experimental/laboratory validation of the elucidated structures is provided.

## Evidence

- [results] top-1 accuracy of 40.76% with competitive similarity metrics, while extreme temperatures degrade performance: [evidence span withheld — anchor status: not_found]
