---
name: quadratic-flux-balance-loss-optimization
description: Use when predicting cell-/sample-wise metabolic flux from scRNA-seq or
  bulk transcriptomics data by training the scFEA factor-graph neural network under
  flux balance constraints, or when reproducing/evaluating its convergence behavior.
when_to_use_negative:
- The goal is reconstruction of context-specific (disease, tissue, or cell type specific)
  metabolic networks and modules — the article lists this as not yet implemented
- Additional biochemical constraints such as redox balance, pH balance, or energy
  balance are required — these are described as future directions, not implemented
- The data are proteomics or metabolomics rather than transcriptomics — flux estimation
  for other omics types is listed as not yet available
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: scFEA
  role: 'Stand-alone implementation of the graph neural network flux solver: instantiates
    per-module networks, builds the quadratic flux-balance and non-negativity losses,
    and trains with Adam in PyTorch'
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web server (scFLUX.org) implementing the scFEA pipeline in a coding-free environment
    for predicting metabolic flux from transcriptomics data
  repo: http://scFLUX.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:a54dd00ade7750a907c7e1f6593781526c1dde27bfe17ce1b0bc4626c49f512d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/changwn/scFEA
  repo_url: https://github.com/changwn/scFEA
  tools:
  - FLUXestimator
  - scFEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# quadratic-flux-balance-loss-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the scFEA/FLUXestimator optimization solver: per-module multi-layer neural networks map gene expression to reaction fluxes, and a quadratic mass-balance loss on intermediate metabolite in-/out-flux plus a non-negativity penalty is minimized with Adam in PyTorch to estimate a cell-wise fluxome.

## When to use

Use when predicting cell-/sample-wise metabolic flux from scRNA-seq or bulk transcriptomics data by training the scFEA factor-graph neural network under flux balance constraints, or when reproducing/evaluating its convergence behavior.

## When NOT to use

- The goal is reconstruction of context-specific (disease, tissue, or cell type specific) metabolic networks and modules — the article lists this as not yet implemented
- Additional biochemical constraints such as redox balance, pH balance, or energy balance are required — these are described as future directions, not implemented
- The data are proteomics or metabolomics rather than transcriptomics — flux estimation for other omics types is listed as not yet available

## Inputs

- Single-cell expression matrix (rows = genes, columns = cells; raw or normalized counts; log transform applied if values exceed 30), e.g. Melissa_full.csv in input_dir
- moduleGene_file mapping metabolic modules to their genes (e.g. module_gene_m168.csv)
- Stoichiometry matrix relating intermediate metabolites (rows) to metabolic modules (columns), e.g. cmMat_171.csv
- Compound name table (cName_file) with compound names and ids
- Human or mouse metabolic module model files from data_dir

## Outputs

- Per-cell, per-module predicted metabolic flux matrix (output_flux_file, e.g. predicted_flux.csv)
- Metabolite stress / imbalance matrix at single-cell resolution (output_balance_file, e.g. metabolite_balance.csv)
- Loss-versus-epoch convergence curve (flux balance loss, non-negative loss, total loss)

## How to apply

Load the single-cell expression matrix from input_dir and the metabolic model files from data_dir with pandas (module_gene_m168.csv for module genes and cmMat_171.csv as the stoichiometry matrix describing intermediate-metabolite to module relationships; use the mouse equivalents for mouse data). Apply MAGIC imputation when the input is 10x-style data (--sc_imputation True). Build a separate multi-layer feed-forward network per metabolic module in PyTorch that maps the module's gene-expression vector to its reaction-flux vector. Define the training objective as a quadratic loss on mass-balance residuals for each intermediate metabolite (produced vs consumed flux across modules) plus a non-negativity penalty on predicted fluxes, with the two loss hyperparameters settable separately, and optimize with Adam. Write per-module predicted fluxes to output_flux_file (e.g. predicted_flux.csv) and residual metabolite balances to output_balance_file (e.g. metabolite_balance.csv). Judge success by plotting loss-vs-epoch curves and confirming the flux balance loss, non-negative loss and total loss decrease and plateau, qualitatively matching supplementary Figure S6.

## Related tools

- **scFEA** (Stand-alone implementation of the graph neural network flux solver: instantiates per-module networks, builds the quadratic flux-balance and non-negativity losses, and trains with Adam in PyTorch) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web server (scFLUX.org) implementing the scFEA pipeline in a coding-free environment for predicting metabolic flux from transcriptomics data) — http://scFLUX.org/

## Examples

```
python src/scFEA.py --input_dir data --res_dir output --test_file Melissa_full.csv
```

## Evaluation signals

- The quadratic flux balance loss, non-negative loss, and total loss all decrease monotonically and plateau during training, matching supplementary Figure S6 convergence behavior
- Output flux and balance matrices are produced with user-defined filenames via output_flux_file and output_balance_file and have one row per cell and one column per metabolic module
- Predicted fluxes are non-negative after training (non-negativity penalty enforced)
- Mass-balance residuals per intermediate metabolite are consistent with the stoichiometry matrix cmMat_171.csv relationships between compounds and modules
- Downstream sanity check: cell clustering on predicted flux reproduces expected metabolic landscapes (e.g. melanoma and head/neck cancer vs stromal cells in the published applications)

## Limitations

- Datasets with more than 10,000 single cells are too time-consuming for the general pipeline without GPU acceleration; the sampling/fitting function (train on subsets per cell type, then fit on all cells) is recommended
- Only flux balance (quadratic loss) constraints are implemented; redox, pH, and energy balance constraints are future work
- Evaluation of the predicted fluxome (goodness of fitting via convergence curves and cross validation) was listed as to-be-released in the README

## Evidence
