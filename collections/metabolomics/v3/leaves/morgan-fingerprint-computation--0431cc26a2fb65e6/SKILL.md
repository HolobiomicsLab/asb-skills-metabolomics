---
name: morgan-fingerprint-computation
description: Use when building a large-scale pretraining corpus of fingerprint–molecule
  pairs from unlabeled molecules (e.g. MassSpecGym's 4M molecules), or when spectra
  must be converted into an experimental-condition-invariant token representation.
when_to_use_negative:
- When annotated experimental spectra with ground-truth structures are already available
  in sufficient quantity — the finetuning stage uses MIST-predicted fingerprints instead
  of recomputing them from structures.
- When the target task requires spectra simulation or fragmentation prediction rather
  than a condition-invariant intermediate representation.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3473
tools:
- name: RDKit
  role: Computes the 4096-bit circular Morgan fingerprints from unlabeled molecules
- name: MIST
  role: Predicts the Morgan fingerprint from a mass spectrum, conditioned on metadata
    and molecular formula, providing spectral input for finetuning
  repo: https://github.com/coleygroup/FRIGID
- name: MS-BART
  role: Consumes fingerprint tokens and molecule (SELFIES) pairs for multi-task pretraining
    and finetuning
  repo: https://github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:0afc4beae2fcca4f60d7178a0eb38e5ae8ca03464e3a3d1f41c7295e00edb642
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/FRIGID
  tools:
  - MIST
  - MS-BART
  - RDKit
  - SMILES
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# morgan-fingerprint-computation

## Summary

Compute 4096-bit circular Morgan fingerprints from molecular structures (or predict them from mass spectra with MIST) so that spectra-free fingerprint–molecule pairs can be built for pretraining MS-BART without simulating spectra under varying collision energies, adduct types, or instrument settings.

## When to use

Use when building a large-scale pretraining corpus of fingerprint–molecule pairs from unlabeled molecules (e.g. MassSpecGym's 4M molecules), or when spectra must be converted into an experimental-condition-invariant token representation.

## When NOT to use

- When annotated experimental spectra with ground-truth structures are already available in sufficient quantity — the finetuning stage uses MIST-predicted fingerprints instead of recomputing them from structures.
- When the target task requires spectra simulation or fragmentation prediction rather than a condition-invariant intermediate representation.

## Inputs

- 4M unlabeled molecules from MassSpecGym (MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv)
- Mass spectra with metadata and chemical formulas (for MIST-predicted fingerprints)
- Molecular structures as SMILES (RDKit input)

## Outputs

- 4096-bit circular Morgan fingerprints FP ∈ {0,1}^4096
- Fingerprint bit tokens of the form <fp{i:04d}>
- Fingerprint–molecule (SELFIES) pairs split into train.tsv and val.tsv under logs/datasets/MassSpecGym/ (e.g. pubchem-4M/)

## How to apply

For unlabeled molecules, compute a 4096-bit circular Morgan fingerprint with RDKit, then turn each activated bit into its <fp{i:04d}> token so fingerprints enter the same 185-token vocabulary as SELFIES strings — fingerprints are invariant to collision energy, adduct, and instrument settings, so pairs can be built without simulating spectra. Optionally apply threshold ϵ to bits for token binarization. To incorporate experimental spectra, predict the fingerprint from the spectrum with the pretrained MIST model, conditioned on metadata and molecular formula, then finetune on those predicted fingerprints to adapt to dataset-specific noise and biases. Output pairs (fingerprint tokens, canonical SELFIES) are written as TSVs, e.g. pubchem-4M/train.tsv and val.tsv after applying MCES ≥ 2 and Tanimoto ≤ 0.5 leakage filters.

## Related tools

- **RDKit** (Computes the 4096-bit circular Morgan fingerprints from unlabeled molecules)
- **MIST** (Predicts the Morgan fingerprint from a mass spectrum, conditioned on metadata and molecular formula, providing spectral input for finetuning) — https://github.com/coleygroup/FRIGID
- **MS-BART** (Consumes fingerprint tokens and molecule (SELFIES) pairs for multi-task pretraining and finetuning) — https://github.com/OpenDFM/MS-BART

## Examples

```
python preprocess/compute_fingerprints.py --input data/MassSpecGym/data/molecules/MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv --output logs/datasets/MassSpecGym/pubchem-4M/train.tsv
```

## Evaluation signals

- Fingerprint dimension must be exactly 4096 bits (circular Morgan radius as used in the paper) and fingerprint tokens must match the <fp{i:04d}> format.
- Downstream pretraining TSVs must exist, parse as tab-separated tables with fingerprint-token and SELFIES columns, and contain no pair violating MCES ≥ 2 (vs. MassSpecGym test fold) or Tanimoto ≤ 0.5 (vs. NPLIB1).
- Model-internal check: fingerprint tokens and SELFIES tokens come from a shared unified vocabulary (185-token SELFIES vocabulary with <fps_sep> separator for hybrid denoising).
- Finetuning stage sanity check: spectra-derived fingerprints produced by MIST must be conditioned on metadata and chemical formula; compare finetuned vs. pretrained loss trajectories to confirm adaptation to dataset noise.

## Limitations

- Accuracy of fingerprints predicted by MIST directly limits downstream structure elucidation performance; the model inherits MIST's errors.
- Fingerprints discard exact fragmentation detail, so substructure-level interpretability from spectra is not recovered by this representation alone.
- The [M+Na]+ adduct class is filtered out before finetuning and alignment, retaining only dominant [M+H]+ spectra — fingerprints computed or predicted for other adducts fall outside the paper's scope.

## Evidence

- [methods] RDKit fingerprint computation from unlabeled molecules: [evidence span withheld — anchor status: not_found]
- [methods] MIST-based spectral fingerprint prediction: [evidence span withheld — anchor status: not_found]
- [methods] Tokenized Morgan fingerprint dimension: [evidence span withheld — anchor status: not_found]
