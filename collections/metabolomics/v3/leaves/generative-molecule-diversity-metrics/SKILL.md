---
name: generative-molecule-diversity-metrics
description: Use when evaluating a diffusion-generated molecular structure set (e.g.
  10,000 molecules from DiffSpectra/DMT reverse denoising on QM9S spectra) to detect
  mode collapse and verify the diversity/accuracy trade-off under temperature sweeps
  or conditioning ablations.
when_to_use_negative:
- When the goal is retrieval-based matching against a fixed library rather than judging
  a generative sampler's diversity - the skill measures generated-set variety, not
  retrieval ranking.
- When spectra are processed one modality in isolation for pure accuracy benchmarking
  (e.g. ir-only or uv-only runs with no diversity/temperature comparison) - diversity
  metrics are most informative across conditioning or tau conditions.
- When evaluating models limited to single-spectrum-type inputs with no generated
  molecule set to tabulate (existing methods process only one spectrum type in isolation
  and are not the target of this skill).
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: DiffSpectra
  role: Source of the generated molecule set being tabulated; spectra-conditioned
    diffusion framework producing samples per tau condition.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: Diffusion Molecule Transformer (DMT)
  role: SE(3)-equivariant denoising network whose sampler settings (temperature tau)
    modulate the diversity/accuracy trade-off being measured.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Spectral encoder whose pre-trained conditioning (pretrained_specformer_path)
    affects the diversity of the conditioned generation being evaluated.
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:4cf71938e0953f5c1d120ca0b0193ed50865aa00edcc5deedda08488bf44dd4a
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# generative-molecule-diversity-metrics

## Summary

Computes diversity metrics (unique atom types, unique bond types, unique SMILES) over a generated molecule set, alongside accuracy metrics, to quantify whether a generative diffusion sampler such as DiffSpectra explores chemically varied structures rather than collapsing to near-duplicates.

## When to use

Use when evaluating a diffusion-generated molecular structure set (e.g. 10,000 molecules from DiffSpectra/DMT reverse denoising on QM9S spectra) to detect mode collapse and verify the diversity/accuracy trade-off under temperature sweeps or conditioning ablations.

## When NOT to use

- When the goal is retrieval-based matching against a fixed library rather than judging a generative sampler's diversity - the skill measures generated-set variety, not retrieval ranking.
- When spectra are processed one modality in isolation for pure accuracy benchmarking (e.g. ir-only or uv-only runs with no diversity/temperature comparison) - diversity metrics are most informative across conditioning or tau conditions.
- When evaluating models limited to single-spectrum-type inputs with no generated molecule set to tabulate (existing methods process only one spectrum type in isolation and are not the target of this skill).

## Inputs

- Generated molecule set from DiffSpectra reverse denoising (e.g. 10,000 samples per temperature, saved via --config.eval.save_mols true)
- Ground-truth reference structures from the QM9S dataset for accuracy attribution
- Per-run sampler configuration (temperature tau, spectra_version: allspectra/ir/raman/uv)

## Outputs

- Diversity counts per run: unique atom types, unique bond types, unique SMILES
- Combined per-temperature table (temperature_sweep.csv) with tau, top1_accuracy, top10_accuracy, unique_atom_types, unique_bond_types, unique_smiles
- Top-1 and top-10 accuracy rates paired with the diversity counts for each sampler setting

## How to apply

For each reverse-time denoising run (starting from a standard Gaussian sample and conditioned on IR/Raman/UV-Vis spectra via SpecFormer embeddings), collect all generated molecules and count unique atom types, unique bond types, and unique SMILES across the set. Aggregate these diversity counts with accuracy metrics (top-1 and top-10 accuracy against ground-truth QM9S references) into a per-condition table, e.g. columns tau, top1_accuracy, top10_accuracy, unique_atom_types, unique_bond_types, unique_smiles. Use the diversity/accuracy pairing to detect degenerate sampling: an extreme noise temperature of tau=0.0 produces overly deterministic samples (39.59% Acc@1, elevated MCES 4.6948) while tau=1.0 gives the best trade-off (40.76% Acc@1). The rationale is that high top-k accuracy alone does not guarantee useful generation; a sampler that emits only a handful of unique SMILES is failing to exploit the spectral conditioning, so diversity counts must be checked in the same pass as accuracy to validate that the tau scaling of injected noise is behaving as intended.

## Related tools

- **DiffSpectra** (Source of the generated molecule set being tabulated; spectra-conditioned diffusion framework producing samples per tau condition.) — https://github.com/AzureLeon1/DiffSpectra
- **Diffusion Molecule Transformer (DMT)** (SE(3)-equivariant denoising network whose sampler settings (temperature tau) modulate the diversity/accuracy trade-off being measured.) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Spectral encoder whose pre-trained conditioning (pretrained_specformer_path) affects the diversity of the conditioned generation being evaluated.) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.ckpts '40' --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- The entry at tau=1.0 in the sweep table reports top-1 accuracy = 40.76%.
- The entry at tau=0.0 reports top-1 accuracy = 39.59% and MCES = 4.6948, confirming the deterministic/low-diversity degenerate regime is captured.
- The sweep table contains rows for tau in {0.0, 0.2, 0.4, 0.6, 0.8, 1.0, 1.2, 1.5, 2.0} sorted ascending by tau.
- Unique_atom_types and unique_bond_types counts increase as tau rises toward 1.0-1.2, consistent with the reported noise-scaling behavior.
- All evaluated runs use --config.eval.num_samples 10000 so diversity counts are computed on an equal sample budget per condition.

## Limitations

- Validation is primarily computational; the diversity/accuracy trade-off on synthetic spectra (QM9S with IR, Raman, UV-Vis) has not been checked experimentally.
- DiffSpectra currently focuses on three spectral types - UV-Vis, IR, and Raman - so diversity conclusions do not extend to NMR or MS modalities.
- Extreme temperatures (tau=0.0 or tau=1.2+) distort both diversity and accuracy, so the counts are only interpretable within the tested sweep range rather than as a universal optimum.

## Evidence

- [results] temperature sweep workflow: [evidence span withheld — anchor status: not_found]
- [results] deterministic regime: [evidence span withheld — anchor status: not_found]
- [results] diversity columns: [evidence span withheld — anchor status: not_found]
- [methods] sampling noise formulation: [evidence span withheld — anchor status: not_found]
