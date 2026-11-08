---
name: metabolic-module-factor-graph-construction
description: Use when a transcriptomics matrix (gene x cell) must be converted to
  cell-wise metabolic reaction fluxes via scFEA's module-wise neural networks coupled
  by a metabolite-by-module stoichiometry (factor-graph) constraint.
when_to_use_negative:
- The goal requires reversible-reaction directionality, inter-cell metabolic exchange/competition,
  or redox/pH/energy balance constraints — these are explicitly listed as not yet
  implemented future work.
- Input data is proteomics or metabolomics rather than transcriptomics — flux estimation
  for other omics types is not yet available.
- A context-specific (disease/tissue/cell-type) metabolic network must be reconstructed
  rather than using the provided curated human or mouse module maps.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2259
tools:
- name: scFEA
  role: Stand-alone Python/PyTorch implementation that builds the module-wise neural
    networks, applies the quadratic flux-balance and non-negativity losses, and solves
    them with Adam
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web server (scFLUX.org) implementing the same scFEA pipeline in a coding-free
    environment for predicting cell-/sample-wise metabolic flux
  repo: http://scFLUX.org/
- name: MAGIC
  role: Imputation of the single-cell expression matrix prior to network training
    (v1.1 fix; recommended True for 10x data)
  repo: https://www.krishnaswamylab.org/projects/magic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:e658cf1c97f8396edf1ba68c95a7ef7effd810c31fac0fd965a04697bd5524a3
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
  - MAGIC
  - scFEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolic-module-factor-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs scFEA's computational metabolic network as a factor graph of focused metabolic modules, each instantiated as its own multi-layer PyTorch neural network mapping module gene expression to reaction-flux rates, with modules coupled through a quadratic mass-balance loss on intermediate metabolites so cell-wise fluxomes can be solved from scRNA-seq data.

## When to use

Use when a transcriptomics matrix (gene x cell) must be converted to cell-wise metabolic reaction fluxes via scFEA's module-wise neural networks coupled by a metabolite-by-module stoichiometry (factor-graph) constraint.

## When NOT to use

- The goal requires reversible-reaction directionality, inter-cell metabolic exchange/competition, or redox/pH/energy balance constraints — these are explicitly listed as not yet implemented future work.
- Input data is proteomics or metabolomics rather than transcriptomics — flux estimation for other omics types is not yet available.
- A context-specific (disease/tissue/cell-type) metabolic network must be reconstructed rather than using the provided curated human or mouse module maps.

## Inputs

- Single-cell expression matrix (CSV, genes as rows and cells as columns; raw or normalized counts, log-transformed if values exceed 30)
- Module-gene table (module_gene_m168.csv default human model; module_gene_mouse equivalent for mouse)
- Stoichiometry matrix (cmMat_171.csv; rows = intermediate metabolites, columns = metabolic modules)
- Compound-name file (compound names and corresponding ids, e.g. cName_c70_m168.csv)

## Outputs

- Predicted cell-wise metabolic flux matrix (one row per cell, one column per metabolic module; output_flux_file, e.g. predicted_flux.csv)
- Metabolite stress/balance matrix at single-cell resolution (output_balance_file, e.g. metabolite_balance.csv)
- Training convergence curves for the flux-balance loss, non-negative loss, and total loss

## How to apply

Load the metabolic model files from data_dir: the module-gene table (module_gene_m168.csv for the default human model, listing the enzymatic genes per module) and the stoichiometry matrix (cmMat_171.csv), whose rows are intermediate metabolites and whose columns are metabolic modules. For 10x-style sparse input, first apply MAGIC imputation (--sc_imputation True), then build one separate multi-layer feed-forward PyTorch network per metabolic module mapping that module's gene-expression input to its reaction-flux vector. Couple the module networks through a factor-graph-style training objective: a quadratic loss on mass-balance residuals (produced vs consumed flux of each intermediate metabolite per the stoichiometry matrix) plus a non-negativity penalty on predicted fluxes, optimized with Adam. The quadratic-loss formulation is deliberate — additional balance/imbalance conditions can be added as further quadratic terms — and hyperparameters of the two losses can be varied as in the paper's four-experiment convergence study. After training, write the per-cell/per-module flux matrix (e.g. predicted_flux.csv via output_flux_file) and the metabolite imbalance matrix (e.g. metabolite_balance.csv via output_balance_file).

## Related tools

- **scFEA** (Stand-alone Python/PyTorch implementation that builds the module-wise neural networks, applies the quadratic flux-balance and non-negativity losses, and solves them with Adam) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web server (scFLUX.org) implementing the same scFEA pipeline in a coding-free environment for predicting cell-/sample-wise metabolic flux) — http://scFLUX.org/
- **MAGIC** (Imputation of the single-cell expression matrix prior to network training (v1.1 fix; recommended True for 10x data)) — https://www.krishnaswamylab.org/projects/magic

## Examples

```
python src/scFEA.py --input_dir data --res_dir output --test_file Melissa_full.csv --moduleGene_file module_gene_m168.csv --stoichiometry_matrix cmMat_171.csv --sc_imputation True
```

## Evaluation signals

- Quadratic flux-balance loss, non-negative loss, and total loss all decrease and plateau over epochs, reproducing the Supplementary Figure S6 convergence behavior (four experiments with different loss hyperparameters).
- Predicted fluxes are non-negative after optimization (the non-negativity penalty term should drive negative values to zero).
- Output flux matrix has one row per input cell and one column per metabolic module matching the module-gene table; the balance matrix dimension matches the intermediate metabolites (rows of cmMat_171.csv).
- Predicted flux and metabolic imbalance are consistent with observed metabolite variation on matched metabolomics data (paper's validation on perturbed oxygen/genetic-condition cells).
- Module Gene/stoichiometry files match the chosen organism model: human defaults are module_gene_m168.csv and cmMat_171.csv; using the mouse model requires the corresponding mouse files released in v1.1.2.

## Limitations

- For datasets with more than 10,000 single cells the general pipeline is too time-consuming without GPU acceleration; a sampling function (train on per-cell-type subsets) plus a fitting function (predict flux on all cells) was planned but not yet released at the time of the README.
- Only mass-balance (quadratic) and non-negativity constraints are implemented; redox, pH, and energy balance are future directions.
- Sample-wise directions of reversible reactions and inter-cell metabolic exchange or competition cannot be assessed.
- MAGIC imputation is recommended (sc_imputation True) for 10x data; a bug in the imputation step was only fixed in v1.1.
- Perturbation analysis and newly curated modules (e.g. methionine, copper ion) are only available in the stand-alone version pending validation.

## Evidence
