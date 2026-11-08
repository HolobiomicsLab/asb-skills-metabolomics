---
name: cancer-metabolic-heterogeneity-analysis
description: Use when scRNA-seq count matrices from tumor microenvironment datasets
  (e.g. melanoma, head-and-neck cancer) need cell-type-resolved metabolic flux quantification,
  especially glucose-glutamine flux comparisons across cancer and stromal cells.
when_to_use_negative:
- The goal is reconstruction of context-specific (disease, tissue, or cell type specific)
  metabolic networks/modules — the tool's authors state this capability does not yet
  exist.
- The input is metabolomics or proteomics data rather than transcriptomics — flux
  estimation for other omics types is not yet available.
- Strict biochemical constraints (redox balance, pH balance, energy balance) must
  be enforced in the flux solution — these are not yet implemented in scFEA.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scFEA
  role: Graph neural network model that infers cell-wise metabolic fluxome and metabolite
    imbalance from imputed scRNA-seq data using focused metabolic modules and flux
    balance constraints
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Webserver (http://scFLUX.org/) implementing the scFEA pipeline in a coding-free
    environment; provides 15 curated human/mouse metabolic networks integrating RECON3D,
    KEGG and literature data
  repo: https://github.com/changwn/scFEA
- name: MAGIC
  role: Dropout imputation of the scRNA-seq expression matrix prior to flux estimation
    (sc_imputation True for 10x data)
  repo: https://www.krishnaswamylab.org/projects/magic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:bc6ece45ef3af60d99c3d08c8b4552ce389c8858fe48ee4e2dd4fbb104ddac1c
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
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# cancer-metabolic-heterogeneity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates cell-wise metabolic flux from tumor microenvironment scRNA-seq using scFEA/FLUXestimator with a Glucose-Glutamine module network (e.g. cName_c70_m168.csv), then compares flux distributions across cancer, myeloid, and T cells to rank glucose metabolic activity (cancer > myeloid > T cells).

## When to use

Use when scRNA-seq count matrices from tumor microenvironment datasets (e.g. melanoma, head-and-neck cancer) need cell-type-resolved metabolic flux quantification, especially glucose-glutamine flux comparisons across cancer and stromal cells.

## When NOT to use

- The goal is reconstruction of context-specific (disease, tissue, or cell type specific) metabolic networks/modules — the tool's authors state this capability does not yet exist.
- The input is metabolomics or proteomics data rather than transcriptomics — flux estimation for other omics types is not yet available.
- Strict biochemical constraints (redox balance, pH balance, energy balance) must be enforced in the flux solution — these are not yet implemented in scFEA.

## Inputs

- Single-cell RNA-seq count matrix (rows = genes, columns = cells; raw or normalized counts, e.g. CSV as in data/melissa full matrix)
- Human metabolic module model files from scFEA data_dir: module_gene_m168.csv, stoichiometry matrix cmMat_171.csv, compound name file cName_c70_m168.csv
- Per-cell cell-type annotations (cancer cell, myeloid cell, T cell)

## Outputs

- Per-cell reaction flux matrix (output_flux_file)
- Per-cell metabolite balance/stress matrix (output_balance_file)
- Pairwise Mann–Whitney U test comparisons of glucose metabolic flux rates across cancer, myeloid, and T cells (threshold P < 0.01)

## How to apply

Load the single-cell count matrices from input_dir and the human metabolic module model files (module_gene_m168.csv, cmMat_171.csv, cName_c70_m168.csv) from data_dir. Impute dropout with MAGIC (magic >= 2.0.4, set --sc_imputation True for 10x data) prior to flux estimation, since dropout distorts enzymatic gene expression driving the flux model. Run the scFEA graph-neural-network flux estimator to predict per-cell reaction fluxes (written to output_flux_file) and metabolite balances/stress (output_balance_file); for datasets >10,000 cells, use the sampling function to train on a per-cell-type subset and the fitting function to predict flux on all cells. Assign each cell its annotated label (cancer cell, myeloid cell, T cell) and compare glucose metabolic flux rate distributions pairwise across the three cell types with the Mann–Whitney U test at significance threshold P < 0.01. Correct application reproduces the ordering: cancer cells highest (lactate production, TCA cycle, nucleotide and serine biosynthesis), then myeloid cells, then T cells, with all pairwise comparisons significant.

## Related tools

- **scFEA** (Graph neural network model that infers cell-wise metabolic fluxome and metabolite imbalance from imputed scRNA-seq data using focused metabolic modules and flux balance constraints) — https://github.com/changwn/scFEA
- **FLUXestimator** (Webserver (http://scFLUX.org/) implementing the scFEA pipeline in a coding-free environment; provides 15 curated human/mouse metabolic networks integrating RECON3D, KEGG and literature data) — https://github.com/changwn/scFEA
- **MAGIC** (Dropout imputation of the scRNA-seq expression matrix prior to flux estimation (sc_imputation True for 10x data)) — https://www.krishnaswamylab.org/projects/magic

## Examples

```
python src/scFEA.py --data_dir data --input_dir input --res_dir output --test_file Melissa_full.csv --moduleGene_file module_gene_m168.csv --stoichiometry_matrix cmMat_171.csv --cName_file cName_c70_m168.csv --sc_imputation True
```

## Evaluation signals

- Predicted flux ordering is reproduced: cancer cells show the highest glucose metabolic rate (lactate production, TCA cycle, nucleotide and serine biosynthesis), followed by myeloid cells, then T cells.
- All pairwise Mann–Whitney U comparisons of glucose flux rates across the three cell types are significant at P < 0.01.
- output_flux_file and output_balance_file are produced with one flux/balance row per input cell and columns matching the metabolic modules.
- Convergence of the flux balance loss and non-negative loss during training (as shown in scFEA convergence diagnostics) indicates a good fit.
- cName_c70_m168.csv contains no NA values (a known fixed bug) and moduleGene/stoichiometry inputs resolve to the m168 human model.

## Limitations

- Datasets with more than 10,000 single cells are too time-consuming for the general pipeline without GPU acceleration; use the sampling and fitting functions instead.
- scFEA cannot yet assess sample-wise directions of reversible reactions or inter-cell metabolic exchange/competition from single cell data.
- Biochemical constraints such as redox, pH, or energy balance are not implemented in the flux estimation.
- Imputation with MAGIC is recommended (set to True) for 10x data; skipping imputation can distort flux estimates on sparse count data.

## Evidence

- [methods] Impute dropout in the scRNA-seq expression matrix with MAGIC prior to flux estimation.: "Fix bugs in single cell imputation step by using [Magic]"
- [abstract] scFEA uses a neural network to estimate reaction rates from transcriptomics data.: "which uses a new neural network architecture to estimate reaction rates from transcriptomics data"
- [results] Cancer cells show the highest glucose metabolic rate followed by myeloid and T cells.: [evidence span withheld — anchor status: not_found]
- [readme] Large datasets require sampling and fitting functions rather than the general pipeline.: "For dataset which contains more than 10,000 single cells, the general pipeline is too time comsuming to get the result without a GPU acceleration."
- [results] Mann–Whitney U test with significance threshold P < 0.01 for pairwise comparisons.: [evidence span withheld — anchor status: not_found]
- [other] Mann–Whitney comparisons are described in the workflow as the statistical test.: [evidence span withheld — anchor status: not_found]
