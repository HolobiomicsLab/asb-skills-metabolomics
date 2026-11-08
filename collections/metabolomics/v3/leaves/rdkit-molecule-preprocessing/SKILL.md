---
name: rdkit-molecule-preprocessing
description: Use when a metabolite or chemical structure dataset arrives as raw SMILES
  strings that need cleanup (salt stripping, charge neutralization, element/token
  filtering) before training or evaluating a chemical language model.
when_to_use_negative:
- Input is already a curated, canonical SMILES set that has previously passed charge-neutralization
  and token filtering — rerunning would risk unnecessary molecule loss.
- Downstream model does not operate on SMILES tokens (e.g., it consumes molecular
  descriptors or graphs directly) and rare-token elimination is irrelevant.
- The goal is not to train/evaluate a chemical language model but rather to generate
  in silico MS/MS libraries from already-generated structures that need no further
  canonicalization.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: Parse SMILES into molecules, remove light fragments, neutralize charges, filter
    valid elements, and canonicalize output SMILES.
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: Runtime environment (>=3.10) in which the clm preprocessing utilities execute.
  repo: https://github.com/skinniderlab/CLM
- name: pip
  role: Installs clm's dependencies via requirements.txt.
- name: uv
  role: Recommended tool to create a virtual environment with the correct Python version
    for the preprocessing pipeline.
  repo: https://github.com/astral-sh/uv
- name: Snakemake
  role: Orchestrates the end-end clm workflow including the preprocessing rule.
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:993539a564ca6d16bbbea771a29d8beb2e9e8d9a67a7fcde7ab12160c6c715f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - Python
  - Python 3.10
  - RDKit
  - Snakemake
  - pip
  - uv
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rdkit-molecule-preprocessing

## Summary

Standardize raw metabolite SMILES into canonical, model-ready strings using RDKit: parse molecules, strip light fragments/salts, neutralize charges, filter for valid elements and rare SMILES vocabulary tokens, then canonicalize for chemical language model training or evaluation.

## When to use

Use when a metabolite or chemical structure dataset arrives as raw SMILES strings that need cleanup (salt stripping, charge neutralization, element/token filtering) before training or evaluating a chemical language model.

## When NOT to use

- Input is already a curated, canonical SMILES set that has previously passed charge-neutralization and token filtering — rerunning would risk unnecessary molecule loss.
- Downstream model does not operate on SMILES tokens (e.g., it consumes molecular descriptors or graphs directly) and rare-token elimination is irrelevant.
- The goal is not to train/evaluate a chemical language model but rather to generate in silico MS/MS libraries from already-generated structures that need no further canonicalization.

## Inputs

- Raw SMILES strings (e.g., HMDB v4.0 per-metabolite SMILES with biological classification and detected/quantified status annotations)
- SMILES vocabulary reference file for rare-token elimination
- Optional metadata columns (e.g., HMDB ID, superclass classification)

## Outputs

- Curated table of canonical SMILES strings with associated metadata
- Per-fold .vocabulary files generated from the canonical SMILES
- Log of molecules removed at each filtering stage (fragments, salts, invalid elements, rare tokens)

## How to apply

Read the input SMILES file and load each string into an RDKit molecule object, removing light fragments and salts/solvents, neutralizing formal charges, and filtering for molecules containing valid chemical elements. Then eliminate molecules containing very rare tokens in the SMILES vocabulary, as these tokens corrupt language model learning. Convert each surviving molecule to its canonical SMILES representation and write it, alongside retained metadata (e.g., HMDB biological classification, detected/quantified status), to a curated table. Finally, use this canonical table as input to training-set splitting (e.g., ten cross-validation folds configured in config.yaml) and vocabulary generation via the clm create_training_sets rule. The rationale is that language models are highly sensitive to token noise: multi-component entries, unneutralized salts, exotic elements, and ultra-rare tokens all degrade the latent structure the model learns from known metabolite chemistry.

## Related tools

- **RDKit** (Parse SMILES into molecules, remove light fragments, neutralize charges, filter valid elements, and canonicalize output SMILES.) — https://github.com/skinniderlab/CLM
- **Python 3.10** (Runtime environment (>=3.10) in which the clm preprocessing utilities execute.) — https://github.com/skinniderlab/CLM
- **pip** (Installs clm's dependencies via requirements.txt.)
- **uv** (Recommended tool to create a virtual environment with the correct Python version for the preprocessing pipeline.) — https://github.com/astral-sh/uv
- **Snakemake** (Orchestrates the end-end clm workflow including the preprocessing rule.) — https://github.com/skinniderlab/CLM

## Examples

```
uv venv && uv pip install -r requirements.txt && snakemake --cores 1 preprocess
```

## Evaluation signals

- Confirm the canonical output file exists and contains a 'smiles' column.
- Verify that every output SMILES parses to a single, neutralized RDKit molecule (no disconnected fragments).
- Check that no output molecule contains invalid or forbidden elements.
- Confirm the ten cross-validation fold files generated downstream are all non-empty.
- Compare the count of removed molecules against the count of rare SMILES vocabulary tokens eliminated to ensure the filter ran to completion.

## Limitations

- Molecules containing very rare tokens in the SMILES vocabulary are eliminated entirely, which may discard legitimate but lexically exotic metabolites.
- Removing salts/solvents and light fragments may strip legitimate multi-component metabolite entries or complexes.
- The filtering thresholds are tuned for a human-metabolite language model; evolutionarily distant applications (e.g., plant or bacterial metabolism) will likely require bespoke re-tuning.
- Pipeline assumes valid input SMILES strings; severely malformed or non-canonical structures may fail RDKit parsing and be silently dropped.

## Evidence

- [other] HMDB v4.0 preprocessing workflow: [evidence span withheld — anchor status: not_found]
- [readme] CLM repository overview: [evidence span withheld — anchor status: external_doc]
