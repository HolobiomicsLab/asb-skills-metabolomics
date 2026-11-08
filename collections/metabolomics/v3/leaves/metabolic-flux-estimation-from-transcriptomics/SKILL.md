---
name: metabolic-flux-estimation-from-transcriptomics
description: Use when single-cell (or bulk) RNA-seq count matrices are available and
  the goal is cell-type-resolved metabolic flux, metabolite concentration changes,
  or comparing flux landscapes (e.g., cancer vs stromal, AD vs control).
when_to_use_negative:
- Input is proteomics or metabolomics data — flux estimation currently extends only
  to transcriptomics; other omics support is listed as future work
- The question requires reversible-reaction directionality or inter-cell metabolic
  exchange/competition — these are not yet implemented by scFEA/FLUXestimator
- Strict biochemical constraints (redox, pH, energy balance) must be enforced — the
  current loss function only supports flux balance and non-negative constraints
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3407
tools:
- name: scFEA
  role: Graph neural network method that estimates cell-wise metabolic flux and metabolite
    balance from imputed scRNA-seq data using curated metabolic modules
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web server (http://scFLUX.org/) implementing the scFEA pipeline coding-free,
    with 15 curated human/mouse metabolic networks integrating RECON3D, KEGG and literature
    data
  repo: https://github.com/changwn/scFEA
- name: MAGIC
  role: Single-cell imputation of the scRNA-seq expression matrix prior to flux estimation
    (magic >= 2.0.4; fixes dropout in the imputation step)
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:50ce758220cd0cd0af1607491772176046c93a1d1b4136e3d6c811d8b1a45fa9
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolic-flux-estimation-from-transcriptomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict cell-wise metabolic flux and metabolite balance/stress from scRNA-seq or bulk transcriptomics using scFEA/FLUXestimator, a graph neural network solver that maps curated metabolic module networks (e.g., m168 human, M171, Glucose-Glutamine) onto gene expression after MAGIC imputation.

## When to use

Use when single-cell (or bulk) RNA-seq count matrices are available and the goal is cell-type-resolved metabolic flux, metabolite concentration changes, or comparing flux landscapes (e.g., cancer vs stromal, AD vs control).

## When NOT to use

- Input is proteomics or metabolomics data — flux estimation currently extends only to transcriptomics; other omics support is listed as future work
- The question requires reversible-reaction directionality or inter-cell metabolic exchange/competition — these are not yet implemented by scFEA/FLUXestimator
- Strict biochemical constraints (redox, pH, energy balance) must be enforced — the current loss function only supports flux balance and non-negative constraints

## Inputs

- Single-cell or single-nucleus RNA-seq count matrix (rows = genes, columns = cells; raw or normalized counts, e.g. CSV)
- Metabolic module model files: moduleGene_file (e.g. module_gene_m168.csv), stoichiometry matrix (e.g. cmMat_171.csv), compound name file (e.g. cName_c70_m168.csv)
- Cell-type annotations for the cells being analyzed

## Outputs

- Per-cell predicted metabolic flux matrix (output_flux_file)
- Per-cell metabolite concentration balance/stress matrix (output_balance_file)
- Downstream tables such as ranked metabolite distinction tables (e.g. metabolite_changes_ad_vs_control.csv) and pairwise cell-type flux comparisons

## How to apply

Load the single-cell count matrix (genes as rows, cells as columns; raw or normalized counts — log transform is applied automatically if values exceed 30) into input_dir and the model files (moduleGene_file e.g. module_gene_m168.csv, stoichiometry_matrix e.g. cmMat_171.csv, cName_file) into data_dir. Impute dropout with MAGIC (set --sc_imputation True, recommended for 10x data) since scFEA's neural network needs denoised expression for enzymatic genes. Run scFEA to predict per-cell reaction fluxes and metabolite balances, writing them via --output_flux_file and --output_balance_file; for datasets with >10,000 cells use the sampling function to train on per-cell-type subsets, then the fitting function to predict flux on all cells (GPU recommended). Downstream, assign reported cell-type labels and compare flux distributions across cell types statistically (e.g., Mann–Whitney U test with P < 0.01) or rank metabolite balance differences between conditions to answer cell-type or disease metabolic questions.

## Related tools

- **scFEA** (Graph neural network method that estimates cell-wise metabolic flux and metabolite balance from imputed scRNA-seq data using curated metabolic modules) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web server (http://scFLUX.org/) implementing the scFEA pipeline coding-free, with 15 curated human/mouse metabolic networks integrating RECON3D, KEGG and literature data) — https://github.com/changwn/scFEA
- **MAGIC** (Single-cell imputation of the scRNA-seq expression matrix prior to flux estimation (magic >= 2.0.4; fixes dropout in the imputation step))

## Examples

```
python src/scFEA.py --input_dir input_dir --data_dir data_dir --res_dir output --moduleGene_file module_gene_m168.csv --stoichiometry_matrix cmMat_171.csv --sc_imputation True
```

## Evaluation signals

- output_flux_file and output_balance_file both exist and are well-formed matrices with one row per cell
- Convergence of the flux balance loss and non-negative loss during training can be inspected (goodness of fitting via convergence curves)
- Reproduced cell-type ordering matches the reported finding, e.g. cancer cells > myeloid cells > T cells in glucose metabolic flux with all pairwise Mann–Whitney U tests significant at P < 0.01
- Ranked AD-vs-control metabolite table contains exactly the expected number of rows (e.g. 14) with reported directionality: increased glycolytic substrates and GABA, decreased glucose and branched-chain amino acids (valine, leucine, isoleucine)
- MAGIC imputation was enabled for 10x-derived data (--sc_imputation True) and model files match the chosen network (e.g. module_gene_m168.csv with cmMat_171.csv)

## Limitations

- Datasets with more than 10,000 cells are too time-consuming for the general pipeline without GPU acceleration; sampling and fitting functions must be used instead
- scFEA's quadratic loss supports flux balance but redox, pH, or energy balance constraints are not implemented
- Context-specific (disease, tissue, cell type) metabolic network reconstruction and flux estimation from proteomics/metabolomics data are future directions, not current capabilities
- Perturbation analysis and newly curated modules (methionine, copper ion) are only available in the stand-alone version pending validation
- Imputation bugs were fixed using MAGIC; the imputation step matters, and cName_c70_m168.csv previously contained NA values (fixed in v1.1.2)

## Evidence
