---
name: top-k-molecular-structure-matching
description: Use when evaluating conditional molecular generators that output multiple
  candidate structures per spectrum and you need rank-based exact-match accuracy (top-k)
  against ground-truth reference structures.
when_to_use_negative:
- Evaluating unconditional molecular generators where each sample has no paired ground-truth
  structure — use distribution-level metrics (FCD, validity) instead
- Comparing generated structures at the 3D conformation level rather than exact structural
  identity — use the RMSD evaluation script (eval_sampled_mols/rmsd.py) instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_0091
tools:
- name: DiffSpectra
  role: Conditional diffusion generator whose eval mode saves sampled molecules and
    computes accuracy/similarity metrics against ground truth
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Pre-trained spectral encoder whose checkpoint presence/absence defines the
    two compared variants in the top-k accuracy table
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:7525bd7890f049d34a6da86dcef5a58a61d347af0d903e1f240f8c0756defc40
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - SpecFormer
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# top-k-molecular-structure-matching

## Summary

Computes top-1 and top-10 structure elucidation accuracy by matching diffusion-generated molecules to ground-truth structures via SMILES identity, the primary accuracy metric for spectra-conditioned generation in DiffSpectra.

## When to use

Use when evaluating conditional molecular generators that output multiple candidate structures per spectrum and you need rank-based exact-match accuracy (top-k) against ground-truth reference structures.

## When NOT to use

- Evaluating unconditional molecular generators where each sample has no paired ground-truth structure — use distribution-level metrics (FCD, validity) instead
- Comparing generated structures at the 3D conformation level rather than exact structural identity — use the RMSD evaluation script (eval_sampled_mols/rmsd.py) instead

## Inputs

- Saved generated molecules from DiffSpectra eval runs (exp/{exp_name}/eval/)
- Ground-truth molecular structures from the QM9S dataset as SMILES
- Per-spectrum candidate rankings (up to 10 generated molecules per spectrum)

## Outputs

- Top-1 accuracy percentage over the test spectra
- Top-10 accuracy percentage over the test spectra
- Similarity metrics table (e.g., specformer_ablation_metrics.csv with per-metric columns for each model variant)

## How to apply

Run the generative model in eval mode (e.g., main.py --mode eval with eval.num_samples 10000 and save_mols true) so generated molecules are saved alongside their conditioning spectra. Compute accuracy and similarity metrics automatically during generation, or manually by running eval_sampled_mols/base_metrics.py, matching each generated molecule to the ground-truth structure by exact SMILES identity rather than fuzzy fingerprint similarity. Tabulate the fraction of test molecules for which the correct structure appears in the top-1 and top-10 ranked candidates. Validate against paper baselines: 40.76% top-1 and 99.49% top-10 accuracy on QM9S with the pre-trained SpecFormer, and ~35.64% top-1 for the untrained SpecFormer ablation.

## Related tools

- **DiffSpectra** (Conditional diffusion generator whose eval mode saves sampled molecules and computes accuracy/similarity metrics against ground truth) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Pre-trained spectral encoder whose checkpoint presence/absence defines the two compared variants in the top-k accuracy table) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
python main.py --config configs/diffspectra_qm9s.py --mode eval --workdir exp/allspectra_pretrained_specformer --config.eval.num_samples 10000 --config.eval.save_mols true --config.data.spectra_version allspectra
```

## Evaluation signals

- Top-1 accuracy for the pre-trained SpecFormer configuration should approach 40.76% on QM9S with 10,000 eval samples
- Delta in top-1 accuracy between pre-trained and random SpecFormer should be approximately +9.49 percentage points
- Top-10 accuracy should be near 99.49%, far above top-1, confirming candidates are ranked with the correct structure among the top 10
- Molecule-to-reference matching must be by exact SMILES identity; check that no fingerprint-similarity threshold was substituted
- Recomputed metrics from base_metrics.py should match the automatic metrics emitted during generation for the same saved molecules

## Limitations

- Matching by SMILES identity means stereochemical or protonation-state differences count as mismatches unless canonicalized identically
- Reported accuracies are on QM9 (small molecules, ≤ heavy-atom constraints of QM9); performance on larger drug-like molecules is not established
- Validation is primarily computational; DiffSpectra currently covers only UV-Vis, IR, and Raman modalities, not NMR or mass spectrometry

## Evidence
