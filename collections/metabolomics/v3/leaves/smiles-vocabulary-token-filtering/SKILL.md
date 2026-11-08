---
name: smiles-vocabulary-token-filtering
description: Use when curating a SMILES training set for a chemical language model
  (e.g., DeepMet/CLM) and you need to eliminate molecules whose strings contain tokens
  too rare to learn reliably.
when_to_use_negative:
- Input is already a canonical, pre-filtered SMILES table that passed preprocessing
  — re-running the rare-token filter adds no value and may drop additional valid rows.
- You are evaluating a model on an external dataset (e.g., PubChem for model evaluation)
  where the vocabulary was fixed at training time; new rare tokens there reflect the
  test distribution, not a training-set cleaning need.
- You are tokenizing a different chemical representation (e.g., SELFIES) where the
  rare-token criterion does not apply as designed.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Converts SMILES to molecule objects, removes light fragments, neutralizes
    charges, and filters for valid elements before rare-token elimination
  repo: https://github.com/skinniderlab/CLM.git
- name: Snakemake
  role: Orchestrates the preprocess rule that reads input SMILES, canonicalizes them,
    and applies the rare-token/molecule-elimination filter in the end-to-end CLM workflow
  repo: https://github.com/skinniderlab/CLM.git
- name: Python 3.10
  role: Runtime environment (>=3.10) in which the CLM preprocessing and vocabulary-filtering
    code executes
  repo: https://github.com/skinniderlab/CLM.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:21c6a9b0f9677adfa3076499d0463107f1434bae5620b129f967ad13f03cd313
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM.git
  tools:
  - DeepMet
  - Python
  - Python 3.10
  - RDKit
  - Snakemake
  - pip
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# SMILES vocabulary token filtering

## Summary

A molecular preprocessing filter that removes molecules containing very rare SMILES tokens from a training corpus, so that the chemical language model's vocabulary is not polluted by infrequent, poorly learned character sequences.

## When to use

Use when curating a SMILES training set for a chemical language model (e.g., DeepMet/CLM) and you need to eliminate molecules whose strings contain tokens too rare to learn reliably.

## When NOT to use

- Input is already a canonical, pre-filtered SMILES table that passed preprocessing — re-running the rare-token filter adds no value and may drop additional valid rows.
- You are evaluating a model on an external dataset (e.g., PubChem for model evaluation) where the vocabulary was fixed at training time; new rare tokens there reflect the test distribution, not a training-set cleaning need.
- You are tokenizing a different chemical representation (e.g., SELFIES) where the rare-token criterion does not apply as designed.

## Inputs

- Raw SMILES strings (e.g., from an HMDB v4.0 XML export) with per-molecule metadata
- RDKit molecule objects after fragment removal/charge neutralization
- SMILES vocabulary frequency table computed over the corpus

## Outputs

- Canonicalized SMILES strings for molecules surviving the rare-token filter, written to a curated table with metadata
- A 'smiles' column in the curated output that downstream create_training_sets can split into folds

## How to apply

As part of the CLM preprocessing pipeline, first convert each input SMILES to an RDKit molecule and perform structural cleanup — removing light fragments and salts/solvents, neutralizing charges, and filtering for valid elements. Then tokenize the remaining canonical SMILES, build a vocabulary frequency table over the corpus, and drop any molecule whose SMILES contains a token flagged as very rare (this also eliminates the corresponding molecules entirely). Retain only canonicalized SMILES strings from surviving molecules plus their metadata. The rationale is that rare tokens cannot be learned reliably by the RNN and would degrade generation quality; filtering them prunes chemically atypical or malformed strings before train/test splitting.

## Related tools

- **RDKit** (Converts SMILES to molecule objects, removes light fragments, neutralizes charges, and filters for valid elements before rare-token elimination) — https://github.com/skinniderlab/CLM.git
- **Snakemake** (Orchestrates the preprocess rule that reads input SMILES, canonicalizes them, and applies the rare-token/molecule-elimination filter in the end-to-end CLM workflow) — https://github.com/skinniderlab/CLM.git
- **Python 3.10** (Runtime environment (>=3.10) in which the CLM preprocessing and vocabulary-filtering code executes) — https://github.com/skinniderlab/CLM.git

## Examples

```
snakemake --cores 1 preprocess
```

## Evaluation signals

- The curated output file exists and contains a 'smiles' column.
- Molecules containing very rare SMILES vocabulary tokens are absent from the surviving table.
- Surviving SMILES strings are canonical (re-canonicalization does not alter them).
- Downstream create_training_sets succeeds: the ten cross-validation fold files are all non-empty.
- Structural-cleanup invariants hold: no light fragments/salts, charges neutralized, all elements valid.

## Limitations

- Removing very rare tokens necessarily eliminates the corresponding molecules, so chemically unusual metabolites can be silently discarded from the training set.
- The filter was designed for human-metabolite corpora; applying it to evolutionarily distant domains (plant or bacterial metabolism) may require a bespoke vocabulary and thresholds.
- The vocabulary is corpus-specific, so a token rare in a small HMDB-derived set may be common in a larger chemical database — the cutoff is not universal.

## Evidence

- [readme] CLM repository provides the Python source for training and evaluating chemical language models.: [evidence span withheld — anchor status: external_doc]
