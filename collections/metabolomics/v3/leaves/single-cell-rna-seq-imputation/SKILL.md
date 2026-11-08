---
name: single-cell-rna-seq-imputation
description: Use when a sparse single-cell RNA-seq count matrix (especially 10x data,
  genes x cells) must be imputed before feeding the scFEA/FLUXestimator neural-network
  flux estimation on the M171 or Glucose-Glutamine module networks.
when_to_use_negative:
- Input is already a cell-level feature table (e.g. predicted fluxes, balances, or
  aggregated metabolite changes) rather than a gene-by-cell count matrix
- Bulk transcriptomics or other omics data types (proteomics, metabolomics) — the
  flux estimation capability is built for single-cell or general transcriptomics,
  and other omics are not yet supported
- Datasets where preserving discrete dropout zeros is critical for the analysis, since
  MAGIC smoothing alters the raw expression distribution
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MAGIC
  role: Performs the single-cell dropout imputation step within the scFEA/FLUXestimator
    pipeline before flux prediction
  repo: https://www.krishnaswamylab.org/projects/magic
- name: scFEA
  role: Consumes the MAGIC-imputed matrix to train the graph neural network that estimates
    cell-wise metabolic flux
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Webserver implementation of the scFEA pipeline, including the Magic-based
    single-cell imputation step
  repo: https://github.com/changwn/scFEA
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:513374e5de07aa3c198a7c312832958689f9573758e6d0928f996b360616351c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/changwn/scFEA
  tools:
  - FLUXestimator
  - MAGIC
  - scFEA
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-rna-seq-imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Denoise and impute dropout in sparse single-cell (or single-nucleus) RNA-seq expression matrices with MAGIC, the imputation step fixed into scFEA v1.1 so that downstream metabolic flux estimation operates on smoothed expression values.

## When to use

Use when a sparse single-cell RNA-seq count matrix (especially 10x data, genes x cells) must be imputed before feeding the scFEA/FLUXestimator neural-network flux estimation on the M171 or Glucose-Glutamine module networks.

## When NOT to use

- Input is already a cell-level feature table (e.g. predicted fluxes, balances, or aggregated metabolite changes) rather than a gene-by-cell count matrix
- Bulk transcriptomics or other omics data types (proteomics, metabolomics) — the flux estimation capability is built for single-cell or general transcriptomics, and other omics are not yet supported
- Datasets where preserving discrete dropout zeros is critical for the analysis, since MAGIC smoothing alters the raw expression distribution

## Inputs

- Single-cell/single-nucleus RNA-seq count matrix (genes x cells; raw or normalized counts, e.g. Melissa_full.csv CSV format)
- scFEA model files from data_dir (module_gene_m168.csv, cmMat_171.csv stoichiometry matrix, cName_c70_m168.csv compound-name table)

## Outputs

- MAGIC-imputed single-cell expression matrix consumed by the scFEA flux estimator
- Downstream per-cell predicted metabolic flux matrix (output_flux_file) and metabolite stress/balance matrix (output_balance_file)

## How to apply

Load the raw or normalized count matrix (rows = genes, columns = cells; logarithm is applied by scFEA if values exceed 30) and run MAGIC (version >= 2.0.4, installed via magic-impute) to fill in dropout, as scFEA v1.1 fixed bugs in the single cell imputation step by using Magic. When invoking the scFEA pipeline, set --sc_imputation True — the README explicitly recommends True for 10x data — so imputation is applied before the graph-neural-network flux estimator trains on the module gene sets (e.g. module_gene_m168.csv, cmMat_171.csv, cName_c70_m168.csv). The rationale is that scFEA's multi-layer networks map enzymatic gene expression to reaction rates, and dropout would otherwise propagate into the predicted fluxome and metabolite balance (stress) matrices. For large datasets (>10,000 cells) where the general pipeline is too slow without a GPU, use the scFEA sampling function on a per-cell-type subset to train and the fitting function to predict flux on all cells after imputation.

## Related tools

- **MAGIC** (Performs the single-cell dropout imputation step within the scFEA/FLUXestimator pipeline before flux prediction) — https://www.krishnaswamylab.org/projects/magic
- **scFEA** (Consumes the MAGIC-imputed matrix to train the graph neural network that estimates cell-wise metabolic flux) — https://github.com/changwn/scFEA
- **FLUXestimator** (Webserver implementation of the scFEA pipeline, including the Magic-based single-cell imputation step) — https://github.com/changwn/scFEA

## Examples

```
python src/scFEA.py --input_dir data --res_dir output --test_file Melissa_full.csv --sc_imputation True
```

## Evaluation signals

- The --sc_imputation flag is set to True for 10x-derived count data, matching the README recommendation
- The imputed matrix retains the original cell dimensionality (one column per input cell) so downstream flux output has one row per input cell
- Downstream scFEA training converges: the flux balance loss, non-negative loss, and total loss decrease and plateau (qualitatively matching supplementary Figure S6 behavior)
- Predicted flux and balance output files exist, are readable tables with finite numeric values for all M171 modules, and reproduce expected cell-type orderings (e.g. cancer > myeloid > T cell glucose flux at Mann–Whitney P < 0.01)

## Limitations

- Datasets with more than 10,000 single cells are too time-consuming for the general pipeline without GPU acceleration; a sampling-then-fitting strategy is needed (still listed as to-be-released in the README)
- Imputation is only wired into the single-cell path of scFEA (--sc_imputation True/False); general (non-single-cell) transcriptomics input follows a different pre-processing route
- No capability yet for context-specific network reconstruction or biochemical constraints (redox, pH, energy balance) in the flux estimation that consumes the imputed data

## Evidence
