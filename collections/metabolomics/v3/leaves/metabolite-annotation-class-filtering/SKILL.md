---
name: metabolite-annotation-class-filtering
description: Use when assembling a curated set of known metabolites from HMDB-classified
  SMILES to train or evaluate a chemical language model and you need to exclude merely
  predicted/expected annotations and lipid-like molecules.
when_to_use_negative:
- The model is intended for human metabolites; applying it to plant or bacterial metabolism
  will require a bespoke model trained on a different curated set.
- Do not use this filter when you want to include lipids or molecules whose biosynthetic
  routes are divergent and undiscovered, since the curated set explicitly excludes
  lipid-like superclasses.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3473
tools:
- name: RDKit
  role: Converts SMILES to molecules and performs light-fragment removal, salt/solvent
    stripping, charge neutralization, element filtering, and canonicalization.
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: Runtime environment for the clm preprocessing utilities and Snakemake workflow.
  repo: https://github.com/skinniderlab/CLM
- name: Snakemake
  role: Runs the end-end clm workflow, including the preprocess and create_training_sets
    steps that implement the class filtering.
  repo: https://github.com/skinniderlab/CLM
- name: pip
  role: Installs clm dependencies from requirements.txt inside a venv or uv-managed
    environment.
  repo: https://github.com/astral-sh/uv
- name: uv
  role: Recommended tool to create a virtual environment with the correct Python version
    for running the clm repository.
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:9751db60788f73fe70f6dce58bd5be1d126c11e5440f45f4948b955011541fcc
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
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolite-annotation-class-filtering

## Summary

Curates a training set of known human metabolites by retaining only HMDB entries annotated in the 'detected' or 'quantified' classes and excluding entries in predicted/expected classes and lipid-like superclasses, then cleaning SMILES with RDKit before model training.

## When to use

Use when assembling a curated set of known metabolites from HMDB-classified SMILES to train or evaluate a chemical language model and you need to exclude merely predicted/expected annotations and lipid-like molecules.

## When NOT to use

- The model is intended for human metabolites; applying it to plant or bacterial metabolism will require a bespoke model trained on a different curated set.
- Do not use this filter when you want to include lipids or molecules whose biosynthetic routes are divergent and undiscovered, since the curated set explicitly excludes lipid-like superclasses.

## Inputs

- HMDB v4.0 XML export with per-metabolite SMILES, biological classification, and detected/quantified status annotations
- config.yaml specifying ten cross-validation folds and train_seeds

## Outputs

- Curated table of canonical SMILES strings plus metadata for 2,046 metabolites experimentally detected in human tissues or biofluids
- Ten per-fold train/test split files with matching .vocabulary files for the training SMILES

## How to apply

Parse the HMDB v4.0 XML export with the clm preprocessing utilities in Python 3.10 + RDKit, extracting per-metabolite SMILES, biological classification, and detected/quantified status. Retain only metabolites in the 'detected' or 'quantified' classes, dropping 'predicted' or 'expected' entries, and exclude molecules whose biological classification falls in the Lipids-and-lipid-like superclass. Convert each SMILES to an RDKit molecule, remove light fragments and salts/solvents, neutralize charges, filter for valid elements, and eliminate molecules containing very rare tokens in the SMILES vocabulary. Canonicalize surviving SMILES and write them with metadata to a curated table, then use the clm 'create_training_sets' rule with ten cross-validation folds configured in config.yaml to split canonical SMILES and generate per-fold .vocabulary files.

## Related tools

- **RDKit** (Converts SMILES to molecules and performs light-fragment removal, salt/solvent stripping, charge neutralization, element filtering, and canonicalization.) — https://github.com/skinniderlab/CLM
- **Python 3.10** (Runtime environment for the clm preprocessing utilities and Snakemake workflow.) — https://github.com/skinniderlab/CLM
- **Snakemake** (Runs the end-end clm workflow, including the preprocess and create_training_sets steps that implement the class filtering.) — https://github.com/skinniderlab/CLM
- **pip** (Installs clm dependencies from requirements.txt inside a venv or uv-managed environment.) — https://github.com/astral-sh/uv
- **uv** (Recommended tool to create a virtual environment with the correct Python version for running the clm repository.) — https://github.com/astral-sh/uv

## Examples

```
snakemake --snakefile Snakefile --configfile config.yaml preprocess create_training_sets
```

## Evaluation signals

- The canonical output file exists, contains a 'smiles' column, and the ten fold files are all non-empty.
- Curated table count matches the expected 2,046 metabolites experimentally detected in human tissues or biofluids.
- No retained entry carries a 'predicted' or 'expected' class annotation, and none maps to the Lipids-and-lipid-like superclass.
- All surviving SMILES survive RDKit parsing, light-fragment removal, charge neutralization, and element filtering without errors.
- Per-fold .vocabulary files are generated for every training fold and are non-empty.

## Limitations

- The language model was trained and evaluated on the structures of human metabolites; evolutionarily distant applications will likely require a bespoke model.
- Inherent limitation anticipating metabolites from divergent, undiscovered biosynthetic routes.
- Predicting MS/MS spectra or retention times from chemical structures remains an open problem, so downstream prioritization cannot rely on those predictions alone.

## Evidence

- [methods] Retain only metabolites annotated in the detected or quantified classes: [evidence span withheld — anchor status: not_found]
- [methods] Exclude lipid-like superclass molecules: [evidence span withheld — anchor status: not_found]
- [results] Training set size and representation: [evidence span withheld — anchor status: not_found]
- [methods] Fold splitting and vocabulary generation: [evidence span withheld — anchor status: not_found]
