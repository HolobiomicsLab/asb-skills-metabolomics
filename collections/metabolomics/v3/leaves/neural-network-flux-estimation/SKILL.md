---
name: neural-network-flux-estimation
description: Use when single-cell (or bulk) transcriptomics data must be converted
  to cell-wise metabolic flux estimates using a flux-balance-constrained neural network,
  e.g. rebuilding the scFEA factor-graph solver or reproducing its loss convergence.
when_to_use_negative:
- The goal is reconstruction of context-specific (disease/tissue/cell-type) metabolic
  networks or modules — this is not yet supported and listed as future work
- Biochemical constraints beyond flux balance (redox, pH, energy balance) or reversible-reaction
  direction and inter-cell exchange assessment are required — not implemented in scFEA/FLUXestimator
- Input is proteomics or metabolomics data — flux estimation is currently restricted
  to transcriptomics input
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: scFEA
  role: 'Stand-alone Python implementation of the flux-balance-constrained neural
    network solver: builds per-module PyTorch networks, minimizes the quadratic mass-balance
    plus non-negativity loss with Adam, and writes flux and balance outputs'
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web-server (http://scFLUX.org/) implementation of the scFEA pipeline providing
    a coding-free environment for the same flux estimation workflow
  repo: https://github.com/changwn/scFEA
- name: MAGIC
  role: Imputation of the single-cell expression matrix performed as the pre-trained
    pipeline's imputation step before network training
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:a96ac20ec23bfe41eea72447f4c26e03730c0dfb1d9a2125eee92a1d6661f3c4
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# neural-network-flux-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the scFEA solver: per-metabolic-module multi-layer neural networks map enzymatic gene expression to reaction fluxes, and a quadratic mass-balance loss plus non-negativity penalty over intermediate metabolites is minimized with Adam in PyTorch to infer cell-wise fluxomes from scRNA-seq data.

## When to use

Use when single-cell (or bulk) transcriptomics data must be converted to cell-wise metabolic flux estimates using a flux-balance-constrained neural network, e.g. rebuilding the scFEA factor-graph solver or reproducing its loss convergence.

## When NOT to use

- The goal is reconstruction of context-specific (disease/tissue/cell-type) metabolic networks or modules — this is not yet supported and listed as future work
- Biochemical constraints beyond flux balance (redox, pH, energy balance) or reversible-reaction direction and inter-cell exchange assessment are required — not implemented in scFEA/FLUXestimator
- Input is proteomics or metabolomics data — flux estimation is currently restricted to transcriptomics input

## Inputs

- Single-cell RNA-seq count or normalized expression matrix (genes x cells, e.g. Melissa_full.csv or 10x data)
- moduleGene file (e.g. module_gene_m168.csv for human; mouse variant available) listing genes per metabolic module
- Stoichiometry matrix (e.g. cmMat_171.csv) mapping intermediate metabolites to metabolic modules
- Compound name file (cName file) with compound names and ids
- Pre-trained module network definitions from data_dir

## Outputs

- Per-cell predicted metabolic flux matrix (output_flux_file, e.g. predicted_flux.csv)
- Per-cell metabolite balance/stress matrix (output_balance_file, e.g. metabolite_balance.csv)
- Loss-versus-epoch convergence curve for flux-balance, non-negativity, and total loss

## How to apply

Load the bundled human example single-cell matrix from input_dir and the metabolic model files from data_dir with pandas: moduleGene file (module_gene_m168.csv for human) defining genes per module, the stoichiometry matrix (cmMat_171.csv) describing intermediate metabolite-to-module relationships, and the compound name file. Apply MAGIC imputation to the expression matrix (recommended for 10x data, sc_imputation=True; log-transform if values exceed 30). Build a factor-graph model in which each of the metabolic modules gets its own multi-layer feed-forward PyTorch network mapping module gene-expression input to the module's reaction-flux vector, capturing the non-linear dependency between enzymatic gene expression and reaction rates. Train with a quadratic loss on mass-balance residuals (produced vs consumed flux per intermediate metabolite via the stoichiometry matrix) plus a non-negativity penalty on predicted fluxes, optimized with Adam; the two losses are weighted by separately tunable hyperparameters. Write per-cell predicted fluxes to output_flux_file and residual metabolite balance/stress to output_balance_file. For datasets above ~10,000 cells without GPU, use the sampling function per cell type followed by the fitting function rather than the joint optimizer.

## Related tools

- **scFEA** (Stand-alone Python implementation of the flux-balance-constrained neural network solver: builds per-module PyTorch networks, minimizes the quadratic mass-balance plus non-negativity loss with Adam, and writes flux and balance outputs) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web-server (http://scFLUX.org/) implementation of the scFEA pipeline providing a coding-free environment for the same flux estimation workflow) — https://github.com/changwn/scFEA
- **MAGIC** (Imputation of the single-cell expression matrix performed as the pre-trained pipeline's imputation step before network training)

## Examples

```
python src/scFEA.py --input_dir data --res_dir output --test_file Melissa_full.csv --moduleGene_file module_gene_m168.csv --stoichiometry_matrix cmMat_171.csv --sc_imputation True
```

## Evaluation signals

- Quadratic flux-balance loss, non-negativity loss, and total loss decrease and plateau over training epochs, reproducing the Supplementary Figure S6 convergence behavior on the pancreatic cancer example
- output_flux_file contains one flux vector per metabolic module per cell consistent with the moduleGene file's module set; output_balance_file residual per intermediate metabolite corresponds to in-/out-flux imbalance via cmMat_171.csv
- Predicted flux and metabolic imbalance are consistent with observed metabolite variation when matched metabolomics data are available (as validated on the perturbed oxygen/genetic condition Pa03c data)
- Downstream sanity checks: cell clustering on predicted flux separates known populations (e.g. cancer vs stromal cells in melanoma and head-and-neck cancer datasets); tSNE of module fluxes for glycolysis/TCA modules is coherent

## Limitations

- The general pipeline is too time-consuming without GPU acceleration for datasets with more than 10,000 single cells; sampling + fitting functions are the recommended alternative
- Only flux balance constraints are implemented; redox, pH, and energy balance or other imbalance conditions are future extensions
- Directionality of reversible reactions and inter-cell metabolic exchange/competition cannot be assessed; context-specific network reconstruction is not yet supported
- Hyperparameter weighting of the flux-balance and non-negativity losses strongly shapes convergence and must be tuned (four weighting configurations shown in Figure S6)

## Evidence
