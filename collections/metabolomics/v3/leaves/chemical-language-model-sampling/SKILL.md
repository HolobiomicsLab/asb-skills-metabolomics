---
name: chemical-language-model-sampling
description: Use when you have one or more trained RNN/CLM checkpoints and need to
  generate a ranked database of candidate chemical structures (with calculated mass,
  formula, and InChIKey) whose sampling frequency can prioritize metabolites for discovery.
when_to_use_negative:
- The input is a set of known metabolites to be identified (this skill generates novel
  candidate structures, not annotations of existing peaks).
- The trained model was fit to evolutionarily distant chemistries (e.g. plant or bacterial
  metabolism) where a bespoke model is required.
- You need MS/MS or retention-time predictions for the generated structures; the article
  notes predicting these from chemical structures remains an open problem.
edam_operation: http://edamontology.org/operation_3985
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_0091
tools:
- name: DeepMet
  role: Trained chemical language model whose learned RNN is sampled autoregressively
    to emit metabolite-like SMILES
  repo: https://github.com/skinniderlab/CLM
- name: PyTorch
  role: Backend used to load the trained RNN checkpoint and run the autoregressive
    sampling loop on GPU
- name: RDKit
  role: Parses sampled SMILES, discards invalid structures, canonicalizes them, and
    computes molecular mass and formula
- name: Snakemake
  role: Orchestrates the end-to-end sampling, validation, and aggregation workflow
    via the Snakefile
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:2283a59f8265bcf8769ca973ba8ad641aac05b291e381ced0ed4370b1eb3fcac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3985
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# chemical-language-model-sampling

## Summary

Sample large numbers of SMILES strings autoregressively from a trained chemical language model (DeepMet/CLM) across multiple folds and seeds, then aggregate per-molecule sampling frequencies to yield a resource of metabolite-like structures ordered by how often the model emits them.

## When to use

Use when you have one or more trained RNN/CLM checkpoints and need to generate a ranked database of candidate chemical structures (with calculated mass, formula, and InChIKey) whose sampling frequency can prioritize metabolites for discovery.

## When NOT to use

- The input is a set of known metabolites to be identified (this skill generates novel candidate structures, not annotations of existing peaks).
- The trained model was fit to evolutionarily distant chemistries (e.g. plant or bacterial metabolism) where a bespoke model is required.
- You need MS/MS or retention-time predictions for the generated structures; the article notes predicting these from chemical structures remains an open problem.

## Inputs

- Trained chemical language model checkpoints (one or more RNN models per cross-validation fold, PyTorch format)
- sample_mols parameter (e.g. 100 million SMILES per fold)
- sample_seed values (distinct seeds for repeated sampling from the same model)
- Fold-specific training-set SMILES files with InChIKeys (for exclusion filtering)

## Outputs

- Tabulated per-molecule sampling frequencies for each fold's generated molecules
- Aggregated structure_frequencies.tsv: one row per unique canonical structure with averaged sampling frequency, calculated mass, and formula
- Set of valid, canonicalized SMILES with InChIKeys, excluding training-set molecules

## How to apply

Load each fold's trained PyTorch RNN and autoregressively sample a large batch of SMILES (e.g. sample_mols=100 million per fold, 1 billion total across ten folds), using distinct sample_seed values so that the same trained model is sampled multiple times. Parse every sampled SMILES with RDKit, discard chemically invalid outputs, canonicalize the rest, and compute each molecule's calculated molecular mass and formula. Exclude any sampled molecule whose InChIKey matches a fold's training-set molecule, and record per-molecule sampling frequency (collect_tabulated_molecules). Finally aggregate across folds using metric='freq-avg' (process_tabulated_molecules) to get one averaged sampling frequency per unique canonical structure, writing results to structure_frequencies.tsv. The sampling frequency acts as a prior: higher-frequency emissions separate withheld metabolites from other generated molecules with near-perfect AUC (~0.98) in cross-validation.

## Related tools

- **DeepMet** (Trained chemical language model whose learned RNN is sampled autoregressively to emit metabolite-like SMILES) — https://github.com/skinniderlab/CLM
- **PyTorch** (Backend used to load the trained RNN checkpoint and run the autoregressive sampling loop on GPU)
- **RDKit** (Parses sampled SMILES, discards invalid structures, canonicalizes them, and computes molecular mass and formula)
- **Snakemake** (Orchestrates the end-to-end sampling, validation, and aggregation workflow via the Snakefile)

## Examples

```
snakemake --config train_seed=0 sample_mols=100000000 sample_seed=42; python process_tabulated_molecules.py --metric freq-avg
```

## Evaluation signals

- structure_frequencies.tsv contains exactly one row per unique canonical structure with its averaged sampling frequency.
- No InChIKey present in the fold's training set appears in the output (training-set molecules are excluded).
- Every emitted SMILES parses as a chemically valid molecule with RDKit; invalid outputs are discarded.
- Cross-validation check: aggregate sampling frequency alone ranks withheld metabolites above other generated molecules with AUC ~0.98.
- Output includes calculated molecular mass and formula for each unique structure.

## Limitations

- The language model was trained and evaluated on human metabolite structures; applying it to evolutionarily distant systems (e.g. plant or bacterial metabolism) will likely require a bespoke model.
- Inherent limitations for anticipating metabolites that originate from divergent, as-of-yet undiscovered biosynthetic routes.
- Only a small fraction of the model's high-frequency emissions may correspond to real metabolites; prospective synthesis or MS/MS integration is still needed for validation.
- Sampling frequencies are a probabilistic prior, not direct evidence of biochemical existence.

## Evidence

- [results] In cross-validation, the aggregate sampling frequency alone separated withheld metabolites from other generated molecules with a near-perfect AUC of 0.98.: [evidence span withheld — anchor status: not_found]
- [methods] Sampling setup: fold-level SMILES generation with per-molecule frequency tabulation and fold aggregation.: [evidence span withheld — anchor status: not_found]
- [methods] Aggregation across folds uses freq-avg to combine per-fold sampling frequencies.: [evidence span withheld — anchor status: not_found]
- [methods] Sampling is repeated with distinct seeds from the same trained model.: [evidence span withheld — anchor status: not_found]
- [methods] SMILES are validated and canonicalized before frequency tabulation.: [evidence span withheld — anchor status: not_found]
