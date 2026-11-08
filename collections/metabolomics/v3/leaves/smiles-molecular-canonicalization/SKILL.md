---
name: smiles-molecular-canonicalization
description: Use when building a training set of chemical structures (e.g., HMDB metabolite
  SMILES) for a chemical language model and a single canonical, noise-free SMILES
  representation per molecule is required before fold splitting.
when_to_use_negative:
- The source metabolite set includes merely predicted or expected HMDB entries rather
  than only detected/quantified ones — those must be filtered out before canonicalization
- Structures belong to the Lipids-and-lipid-like superclass and the target is the
  DeepMet known-metabolome model, which excludes them
- The downstream task is MS/MS spectral annotation rather than structure-based language
  model training — canonical SMILES alone do not prioritize unknown peaks
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Converts SMILES to molecules, removes light fragments, neutralizes charges,
    filters for valid elements, and produces the canonical SMILES strings
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: Runtime for the clm preprocessing utilities that parse input SMILES and write
    canonical output
  repo: https://github.com/skinniderlab/CLM
- name: Snakemake
  role: Runs the preprocess and create_training_sets rules of the clm Snakefile end-to-end,
    including the ten-fold splits and vocabulary files
  repo: https://github.com/skinniderlab/CLM
- name: pip
  role: Installs the clm repository's dependencies via requirements.txt in a virtual
    environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:c9793a80ae8df324aae864ed433c87770bd593900c947441b19eb7871fb0df2e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# smiles-molecular-canonicalization

## Summary

Standardizes metabolite structures into canonical SMILES strings as the structural representation used to train the DeepMet chemical language model, after RDKit-based cleanup of fragments, charges, elements, and rare SMILES tokens.

## When to use

Use when building a training set of chemical structures (e.g., HMDB metabolite SMILES) for a chemical language model and a single canonical, noise-free SMILES representation per molecule is required before fold splitting.

## When NOT to use

- The source metabolite set includes merely predicted or expected HMDB entries rather than only detected/quantified ones — those must be filtered out before canonicalization
- Structures belong to the Lipids-and-lipid-like superclass and the target is the DeepMet known-metabolome model, which excludes them
- The downstream task is MS/MS spectral annotation rather than structure-based language model training — canonical SMILES alone do not prioritize unknown peaks

## Inputs

- HMDB v4.0 XML export with per-metabolite SMILES and detected/quantified status annotations
- Raw (non-canonical) SMILES strings file for the clm preprocess step
- config.yaml specifying the number of cross-validation folds (ten) and train_seeds

## Outputs

- Canonical SMILES file (curated table with a 'smiles' column plus metabolite metadata)
- Per-fold train/test split files with accompanying .vocabulary files for the training SMILES
- Optional augmented SMILES enumerations for data augmentation

## How to apply

Parse the source structure export (e.g., the HMDB v4.0 XML dump) to extract per-metabolite SMILES, retaining only metabolites annotated as detected or quantified (not merely predicted or expected) and dropping molecules whose biological classification maps to the Lipids-and-lipid-like superclass. Convert each SMILES to an RDKit molecule and remove light fragments and salts/solvents, neutralize charges, filter for valid elements, and eliminate molecules containing very rare tokens in the SMILES vocabulary. Canonicalize each surviving SMILES and write the canonical strings with their metadata to a curated table. Optionally augment SMILES with an enum factor, and optionally apply a min_tc parameter to keep only mutually similar SMILES. Finally, run the create_training_sets rule with ten cross-validation folds configured in config.yaml to split the canonical SMILES and emit per-fold .vocabulary files. Canonicalization rationale: the RNN consumes a tokenized SMILES vocabulary, so inconsistent atom ordering and non-canonical syntax would fragment the vocabulary and degrade generation.

## Related tools

- **RDKit** (Converts SMILES to molecules, removes light fragments, neutralizes charges, filters for valid elements, and produces the canonical SMILES strings) — https://github.com/skinniderlab/CLM
- **Python 3.10** (Runtime for the clm preprocessing utilities that parse input SMILES and write canonical output) — https://github.com/skinniderlab/CLM
- **Snakemake** (Runs the preprocess and create_training_sets rules of the clm Snakefile end-to-end, including the ten-fold splits and vocabulary files) — https://github.com/skinniderlab/CLM
- **pip** (Installs the clm repository's dependencies via requirements.txt in a virtual environment)

## Examples

```
snakemake --cores 1 preprocess && snakemake --cores 1 create_training_sets
```

## Evaluation signals

- The canonical output file exists and contains a 'smiles' column
- Every canonical SMILES re-parses in RDKit and re-canonicalizes to an identical string (idempotence check)
- No surviving molecule contains invalid elements, un-neutralized charges, light fragments/salts, or very rare SMILES vocabulary tokens
- All ten cross-validation fold files produced by create_training_sets are non-empty and each has a corresponding .vocabulary file
- Lipids-and-lipid-like superclass entries and predicted/expected-only HMDB records are absent from the curated table

## Limitations

- Removing very rare tokens in the SMILES vocabulary eliminates the corresponding molecules entirely, so legitimately unusual metabolites may be dropped
- The trained model was built and evaluated on human metabolite structures; transfer to evolutionarily distant metabolism (plant, bacterial) likely requires a bespoke model and re-canonicalized training set
- SMILES augmentation via the enum factor is optional and typically applied, so results differ if augmentation is disabled

## Evidence

- [readme] CLM repository purpose for training chemical language models: [evidence span withheld — anchor status: external_doc]
