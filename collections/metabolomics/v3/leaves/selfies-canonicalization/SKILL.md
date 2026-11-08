---
name: selfies-canonicalization
description: Use when building molecule–fingerprint training pairs for a mass-spectra
  language model and the molecular targets must be a uniform, robust string encoding
  of structures.
when_to_use_negative:
- The downstream model consumes SMILES directly and does not share a unified SELFIES
  token vocabulary
- You only need molecular fingerprints (e.g. 4096-bit Morgan from RDKit) and no molecular
  sequence target
- The task requires experimental spectra simulation rather than structure string targets
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: SELFIES
  role: Produces the canonical SELFIES string representation of each molecule used
    as the translation target
- name: RDKit
  role: Computes the 4096-bit circular Morgan fingerprint paired with each canonical
    SELFIES string
- name: MS-BART
  role: Model whose unified vocabulary consumes the canonical SELFIES sequences and
    fingerprint tokens during multi-task pretraining
  repo: https://github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:1b4bc36032037ca764064d77d4bdd93c97230c4ac0bb88b41cac6534eadc7687
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# selfies-canonicalization

## Summary

Canonicalize molecular structures into SELFIES string representations with a fixed token vocabulary (~185 tokens) so that fingerprint–molecule pretraining pairs share one molecular sequence format across the corpus.

## When to use

Use when building molecule–fingerprint training pairs for a mass-spectra language model and the molecular targets must be a uniform, robust string encoding of structures.

## When NOT to use

- The downstream model consumes SMILES directly and does not share a unified SELFIES token vocabulary
- You only need molecular fingerprints (e.g. 4096-bit Morgan from RDKit) and no molecular sequence target
- The task requires experimental spectra simulation rather than structure string targets

## Inputs

- Molecular structure records (e.g. SMILES strings) from a TSV table such as MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv

## Outputs

- Canonical SELFIES string per molecule (185-token vocabulary), paired with fingerprint tokens <fp{i:04d}> as train/val rows

## How to apply

Convert each molecular structure (e.g. from the 4M unlabeled molecule table MassSpecGym_molecules_MCES2_disjoint_with_test_fold_4M.tsv) into its canonical SELFIES string, a string-based molecular representation alongside SMILES with a bounded 185-token vocabulary. The resulting SELFIES sequence S={s1,…,sl} is the decoder target for pretraining tasks such as Fingerprint-to-Molecule Translation and SELFIES Denoising, where 30% of tokens are randomly masked with a Mask token. Convert each molecule's 4096-bit circular Morgan fingerprint (computed with RDKit) into <fp{i:04d}> tokens and pair it with the canonical SELFIES string to form the pretraining rows. Rationale: SELFIES provides a robust, tokenized molecular representation that avoids SMILES parsing errors and enables the same vocabulary to be shared between fingerprint tokens and molecular tokens in a unified vocab.

## Related tools

- **SELFIES** (Produces the canonical SELFIES string representation of each molecule used as the translation target)
- **RDKit** (Computes the 4096-bit circular Morgan fingerprint paired with each canonical SELFIES string)
- **MS-BART** (Model whose unified vocabulary consumes the canonical SELFIES sequences and fingerprint tokens during multi-task pretraining) — https://github.com/OpenDFM/MS-BART

## Examples

```
from selfies import encoder; selfies_str = encoder(smiles_str)  # canonical SELFIES for each molecule in the 4M corpus
```

## Evaluation signals

- Output SELFIES strings decode to the same molecule as the input SMILES
- All generated SELFIES strings belong to the expected 185-token vocabulary
- Each pretraining row contains one fingerprint-token sequence paired with exactly one SELFIES sequence
- Denosing corruption replaces exactly 30% of SELFIES tokens with the Mask token and the recovered sequence matches the original

## Limitations

- SELFIES sequences are variable length (S={s1,…,sl}), so masking and translation must handle per-sequence lengths rather than fixed-size tensors
- The article reports no simulation of mass spectra for the 4M unlabeled molecules, so fingerprint–SELFIES pairs come from computed fingerprints only; coverage is limited to structures present in the source table
- The MIST-predicted fingerprints used at finetuning time differ in reliability from the RDKit-computed fingerprints used in pretraining, so SELFIES targets learned pretraining may be conditioned on noisier inputs

## Evidence

- [methods] SELFIES token vocabulary and masking objective: [evidence span withheld — anchor status: not_found]
- [methods] Fingerprint-to-molecule translation target: [evidence span withheld — anchor status: not_found]
- [methods] SELFIES as string representation: [evidence span withheld — anchor status: not_found]
- [methods] Fingerprint computation for unlabeled molecules: [evidence span withheld — anchor status: not_found]
- [intro] Pretraining corpus construction: [evidence span withheld — anchor status: not_found]
- [methods] Pretrain data leakage filter: [evidence span withheld — anchor status: not_found]
