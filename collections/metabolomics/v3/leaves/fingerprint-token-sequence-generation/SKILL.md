---
name: fingerprint-token-sequence-generation
description: Use when building a large-scale pretraining corpus of fingerprint–molecule
  pairs from unlabeled molecular structures, where fingerprints must be expressed
  as tokens in a unified vocabulary rather than raw bit vectors.
when_to_use_negative:
- Input is already experimental mass spectra paired with annotated structures — this
  skill generates pretraining data from unlabeled structures, not from simulated or
  measured spectra.
- The downstream model expects raw 4096-bit fingerprint vectors rather than token
  sequences in a unified vocabulary.
- The candidate molecules violate the leakage constraints (MCES distance < 2 to test-fold
  molecules or Tanimoto > 0.5 to NPLIB1) and cannot be filtered.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: Computes the 4096-bit circular Morgan fingerprints from unlabeled molecular
    structures
- name: SELFIES
  role: Provides the robust string-based molecular target representation paired with
    each fingerprint-token sequence
- name: MS-BART
  role: Downstream model pretrained on the generated fingerprint–molecule pairs using
    multi-task objectives
  repo: https://github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:498978d545b4529df443db3f0adb2c02d2a055656534225f10daa262b72dc364
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - MS-BART
  - RDKit
  - SELFIES
  - SMILES
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fingerprint-token-sequence-generation

## Summary

Converts unlabeled molecules into pretraining-ready fingerprint–molecule pairs by rendering each Morgan fingerprint bit as a dedicated vocabulary token and pairing it with a canonical SELFIES target sequence, enabling cross-modal pretraining without simulating mass spectra.

## When to use

Use when building a large-scale pretraining corpus of fingerprint–molecule pairs from unlabeled molecular structures, where fingerprints must be expressed as tokens in a unified vocabulary rather than raw bit vectors.

## When NOT to use

- Input is already experimental mass spectra paired with annotated structures — this skill generates pretraining data from unlabeled structures, not from simulated or measured spectra.
- The downstream model expects raw 4096-bit fingerprint vectors rather than token sequences in a unified vocabulary.
- The candidate molecules violate the leakage constraints (MCES distance < 2 to test-fold molecules or Tanimoto > 0.5 to NPLIB1) and cannot be filtered.

## Inputs

- Unlabeled molecule table TSV (e.g. MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv, 4M molecules)
- Molecular structures as SMILES strings

## Outputs

- pubchem-4M/train.tsv and pubchem-4M/val.tsv under logs/datasets/MassSpecGym/, each row a fingerprint-token sequence paired with a SELFIES molecule string
- 10,000-entry validation split reserved for pretraining evaluation

## How to apply

Load the unlabeled molecule table (e.g. data/MassSpecGym/data/molecules/MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv, 4 million molecules) and compute a 4096-bit circular Morgan fingerprint for each structure with RDKit. Convert each active bit into its dedicated <fp{i:04d}> token form so that fingerprints and molecular strings share one vocabulary, and render each molecule as a canonical SELFIES string drawn from the 185-token SELFIES vocabulary. Apply the leakage filters described in the paper — exclude molecules with MCES distance less than two from any MassSpecGym test-fold molecule and molecules with Tanimoto similarity greater than 0.5 to any NPLIB1 molecule — before splitting. Finally, split the fingerprint-token/SELFIES pairs with preprocess/split_pretrain_dataset.py, reserving 10,000 entries for validation, so the pairs can drive the multi-task pretraining objectives (SELFIES denoising, fingerprint-to-molecule translation, hybrid denoising).

## Related tools

- **RDKit** (Computes the 4096-bit circular Morgan fingerprints from unlabeled molecular structures)
- **SELFIES** (Provides the robust string-based molecular target representation paired with each fingerprint-token sequence)
- **MS-BART** (Downstream model pretrained on the generated fingerprint–molecule pairs using multi-task objectives) — https://github.com/OpenDFM/MS-BART

## Examples

```
python preprocess/split_pretrain_dataset.py --input data/MassSpecGym/data/molecules/MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv --out logs/datasets/MassSpecGym/pubchem-4M/
```

## Evaluation signals

- The produced train.tsv and val.tsv files exist and parse as tab-separated tables with a fingerprint-token column and a SELFIES column.
- No fingerprint-token/SELFIES pair violates the leakage constraints: MCES distance must be >= 2 from any MassSpecGym test-fold molecule and Tanimoto similarity <= 0.5 to any NPLIB1 molecule.
- Fingerprint tokens follow the <fp{i:04d}> format and SELFIES strings decode within the 185-token SELFIES vocabulary.
- The validation split contains exactly 10,000 entries.
- Downstream pretraining on the generated pairs improves multi-task metrics (SELFIES denoising, fingerprint-to-molecule translation, hybrid denoising) relative to training without pretraining.

## Limitations

- The MCES-distance leakage filter requires computing maximum common edge substructure distances against all test-fold molecules, which can be expensive at 4M scale.
- The generated pretraining pairs are fingerprints of computed structures, not simulated spectra; the approach assumes fingerprints are invariant to experimental conditions such as collision energies, adduct types, and instrument settings.
- The fingerprint-token vocabulary is tied to the 4096-bit Morgan fingerprint and the <fp{i:04d}> token scheme; a different fingerprint definition requires regenerating the vocabulary.

## Evidence

- [intro] fingerprint tokens from 4M unlabeled molecules: [evidence span withheld — anchor status: not_found]
- [methods] SELFIES as molecular representation: [evidence span withheld — anchor status: not_found]
- [methods] pretraining objectives: [evidence span withheld — anchor status: not_found]
