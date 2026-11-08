---
name: neural-network-flux-balance-modeling
description: Use when a single-cell count matrix (genes x cells, raw or normalized
  counts) needs cell-wise reaction-rate (fluxome) prediction and metabolite-variation
  estimates, e.g., comparing glucose-glutamine flux among cancer, myeloid, and T cells
  in tumor microenvironment data.
when_to_use_negative:
- Input is already a flux or feature table (e.g., pre-computed ssGSEA module scores)
  rather than a gene-by-cell count matrix.
- The analysis requires context-specific (disease/tissue/cell-type-specific) metabolic
  network reconstruction, reversible reaction directionality, inter-cell metabolic
  exchange, or biochemical constraints such as redox, pH, or energy balance — these
  are explicitly not yet supported.
- Input is proteomics or metabolomics data — flux estimation currently supports transcriptomics
  only.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2259
tools:
- name: scFEA
  role: Stand-alone graph neural network pipeline that estimates cell-wise metabolic
    flux and metabolite stress from imputed scRNA-seq against the M171 module structure;
    writes output_flux_file and output_balance_file.
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web server (http://scFLUX.org/) implementing the scFEA pipeline in a coding-free
    environment with 15 curated human/mouse metabolic networks integrating RECON3D,
    KEGG and literature data.
  repo: https://github.com/changwn/scFEA
- name: MAGIC
  role: Performs single-cell dropout imputation of the count matrix prior to flux
    estimation (fixed in scFEA v1.1; magic >= 2.0.4).
  repo: https://www.krishnaswamylab.org/projects/magic
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:08e77e7120c7bd1620169b4ed5b0a31b93d1ef62732cb5336e979be028ed9205
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# neural-network-flux-balance-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates cell-wise metabolic flux and metabolite balance (stress) from scRNA-seq or bulk transcriptomics using scFEA/FLUXestimator, a graph neural network solver that enforces flux balance constraints over a curated human metabolic module map (e.g., M171 modules), enabling analysis of metabolic heterogeneity across cell types.

## When to use

Use when a single-cell count matrix (genes x cells, raw or normalized counts) needs cell-wise reaction-rate (fluxome) prediction and metabolite-variation estimates, e.g., comparing glucose-glutamine flux among cancer, myeloid, and T cells in tumor microenvironment data.

## When NOT to use

- Input is already a flux or feature table (e.g., pre-computed ssGSEA module scores) rather than a gene-by-cell count matrix.
- The analysis requires context-specific (disease/tissue/cell-type-specific) metabolic network reconstruction, reversible reaction directionality, inter-cell metabolic exchange, or biochemical constraints such as redox, pH, or energy balance — these are explicitly not yet supported.
- Input is proteomics or metabolomics data — flux estimation currently supports transcriptomics only.

## Inputs

- Single-cell expression count matrix (rows = genes, columns = cells; raw or normalized counts, e.g., Melissa_full.csv)
- scFEA human metabolic module model files: module_gene_m168.csv, cmMat_171.csv stoichiometry matrix, cName_c70_m168.csv compound names
- Cell-type annotations for downstream flux comparison (e.g., cancer cell, myeloid cell, T cell)

## Outputs

- Predicted cell-wise metabolic flux matrix (one row per cell, one column per M171 module/reaction flux)
- Metabolite balance / stress matrix (metabolite variation at single-cell resolution)
- Downstream: cell-type-specific flux distributions with pairwise test statistics (e.g., Mann-Whitney U, P < 0.01)

## How to apply

Load the metabolic module model files (module_gene_m168.csv, cmMat_171.csv stoichiometry, and cName compound files such as cName_c70_m168.csv) from data_dir and the count matrix from input_dir; use module_gene_m168.csv and cmMat_171.csv for the human model. Impute dropout with MAGIC (magic >= 2.0.4; sc_imputation=True recommended for 10x data), log-transforming only if values exceed 30. Run the scFEA neural-network estimator (torch-based graph neural network) to predict per-cell flux across all M171 modules, writing predicted flux to output_flux_file and metabolite balance/stress to output_balance_file. For datasets over 10,000 cells, the general pipeline is too slow without GPU; use the sampling function to train on a subset per cell type, then the fitting function to predict flux on all cells. Downstream, assign reported cell-type labels and compare flux distributions pairwise (e.g., Mann-Whitney U test, significance threshold P < 0.01) to rank cell types such as cancer vs myeloid vs T cells by glucose metabolic rate.

## Related tools

- **scFEA** (Stand-alone graph neural network pipeline that estimates cell-wise metabolic flux and metabolite stress from imputed scRNA-seq against the M171 module structure; writes output_flux_file and output_balance_file.) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web server (http://scFLUX.org/) implementing the scFEA pipeline in a coding-free environment with 15 curated human/mouse metabolic networks integrating RECON3D, KEGG and literature data.) — https://github.com/changwn/scFEA
- **MAGIC** (Performs single-cell dropout imputation of the count matrix prior to flux estimation (fixed in scFEA v1.1; magic >= 2.0.4).) — https://www.krishnaswamylab.org/projects/magic

## Examples

```
python src/scFEA.py --input_dir data --res_dir output --test_file Melissa_full.csv --moduleGene_file module_gene_m168.csv --stoichiometry_matrix cmMat_171.csv --sc_imputation True
```

## Evaluation signals

- Both output files exist and are readable tables with one row per input cell and one column per predicted flux/balance variable, with finite numeric estimates for all M171 modules.
- Loss curves: flux balance loss and non-negative loss converge during training (check convergence curves as in scFEA Figure S6).
- On the melanoma/head-and-neck tumor microenvironment benchmark with the Glucose-Glutamine network, the reproduced ordering is cancer cells > myeloid cells > T cells in glucose metabolic flux, with all pairwise Mann-Whitney comparisons significant at P < 0.01.
- Predicted flux and metabolite imbalance are consistent with observed variation in matched metabolomics data (as validated on the perturbed oxygen/genetic condition scRNA-seq dataset).
- The cName_c70_m168.csv compound file contains no NA values (bug fixed in scFEA v1.1.2).

## Limitations

- The general pipeline is too time-consuming for datasets with more than 10,000 single cells without GPU acceleration; sampling and fitting functions for large datasets were still pending release.
- No capability yet for reconstruction of context-specific (disease, tissue, cell type specific) metabolic networks and modules.
- No capability yet for sample-wise assessment of reversible reaction directions and inter-cell metabolic exchange or competition from single cell data.
- Biochemical constraints such as redox balance, pH balance, or energy balance are not yet implemented in the flux estimation method.
- Flux estimation for other omics data types (proteomics, metabolomics) is not yet available; perturbation analysis and newly curated modules (methionine, copper ion) are only in the stand-alone version pending web server validation.

## Evidence
