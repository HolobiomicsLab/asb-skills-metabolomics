---
name: cross-fold-sampling-frequency-aggregation
description: Use when multiple cross-validation folds or seed-replicated generative
  RNN models have each produced per-molecule sampling-frequency tables that must be
  merged into one averaged frequency table for ranking candidate metabolites.
when_to_use_negative:
- Only a single model or single fold exists, so there is nothing to aggregate across
  and no held-out structure to validate frequency against
- The goal is identifying known metabolites already in the training set — training-set
  InChIKeys are deliberately excluded here, so known-compound recall cannot be assessed
- Input data are experimental spectra or annotations rather than generated SMILES
  with sampling frequencies
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: PyTorch
  role: Runs the trained RNN autoregressive SMILES sampling per fold and seed
- name: RDKit
  role: Parses and validates sampled SMILES, canonicalizes structures, computes mass/formula
    and InChIKeys for training-set filtering
- name: Snakemake
  role: Orchestrates the end-to-end workflow including collect_tabulated_molecules
    and process_tabulated_molecules aggregation steps
- name: DeepMet
  role: The chemical language model whose generated structures and sampling frequencies
    are being aggregated
  repo: https://github.com/skinniderlab/CLM.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:3cd55e54e54183335db256cbace69e5fb0a76e821eda72f1cb0530b4e56a20bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM.git
  tools:
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# cross-fold-sampling-frequency-aggregation

## Summary

Aggregates per-molecule sampling frequencies of autoregressively generated SMILES across all trained RNN models and cross-validation folds of DeepMet (1 billion sampled SMILES total), producing a single tabulation of unique canonical structures ranked by average sampling frequency for metabolite prioritization.

## When to use

Use when multiple cross-validation folds or seed-replicated generative RNN models have each produced per-molecule sampling-frequency tables that must be merged into one averaged frequency table for ranking candidate metabolites.

## When NOT to use

- Only a single model or single fold exists, so there is nothing to aggregate across and no held-out structure to validate frequency against
- The goal is identifying known metabolites already in the training set — training-set InChIKeys are deliberately excluded here, so known-compound recall cannot be assessed
- Input data are experimental spectra or annotations rather than generated SMILES with sampling frequencies

## Inputs

- Trained RNN model checkpoints (PyTorch) for each of ten cross-validation folds
- Per-fold training-set SMILES (for InChIKey-based exclusion of training molecules)
- config.yaml specifying sample_mols per fold, sample_seed values, and |train_seeds|
- Raw sampled SMILES strings from autoregressive generation

## Outputs

- structure_frequencies.tsv: one row per unique canonical structure with its averaged sampling frequency
- Per-molecule calculated molecular mass and formula annotations
- Cross-validation separation metric of withheld metabolites vs other generated molecules (AUC 0.98 reported)

## How to apply

Load each fold's trained RNN model (PyTorch) and autoregressively sample the configured number of SMILES per fold (100 million per fold, 1 billion total across ten folds), using distinct sample_seed values to resample from the same trained model. Parse every sampled SMILES with RDKit, discard chemically invalid outputs, canonicalize the valid ones, and compute molecular mass and formula. Remove any sampled molecule whose InChIKey matches the fold's training set, then record the per-molecule sampling frequency. Aggregate the tabulated molecules across all |train_seeds| models within each fold via collect_tabulated_molecules, then merge across all folds with process_tabulated_molecules using metric=freq-avg to obtain the average sampling frequency per unique structure. The rationale is that structures sampled repeatedly by independent models held out from them are more likely to represent real, discoverable metabolites.

## Related tools

- **PyTorch** (Runs the trained RNN autoregressive SMILES sampling per fold and seed)
- **RDKit** (Parses and validates sampled SMILES, canonicalizes structures, computes mass/formula and InChIKeys for training-set filtering)
- **Snakemake** (Orchestrates the end-to-end workflow including collect_tabulated_molecules and process_tabulated_molecules aggregation steps)
- **DeepMet** (The chemical language model whose generated structures and sampling frequencies are being aggregated) — https://github.com/skinniderlab/CLM.git

## Examples

```
snakemake --cores 8 collect_tabulated_molecules && snakemake --cores 8 process_tabulated_molecules
```

## Evaluation signals

- structure_frequencies.tsv contains exactly one row per unique canonical structure, with an averaged (freq-avg) sampling frequency value
- No training-set InChIKeys from any fold appear in the final table
- No chemically invalid SMILES are present in the output (all were parsed with RDKit and invalid ones discarded)
- The aggregate sample size matches the configuration: sample_mols SMILES per fold summed over ten folds equals 1 billion
- Withheld metabolites separate from other generated molecules by aggregate sampling frequency with near-perfect AUC (0.98) in cross-validation

## Limitations

- Sampling frequency reflects structures generated by models trained on known human metabolites; evolutionarily distant applications (e.g., plant or bacterial metabolism) likely require bespoke models, so frequency-based ranking may not transfer
- Inherent limitation in anticipating metabolites from divergent, as-of-yet undiscovered biosynthetic routes — low-frequency truly novel structures may be systematically under-ranked
- Excluding training-set InChIKeys means the table quantifies generalization to putatively novel structures only, not reproduction of known metabolites

## Evidence

- [results] AUC validation: [evidence span withheld — anchor status: not_found]
