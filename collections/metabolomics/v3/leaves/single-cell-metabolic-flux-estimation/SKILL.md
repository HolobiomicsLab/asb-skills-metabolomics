---
name: single-cell-metabolic-flux-estimation
description: Use when a single-cell expression matrix (rows = genes, columns = cells,
  raw or normalized counts, e.g. Melissa_full.csv) needs cell-wise predicted metabolic
  flux and metabolite variation outputs from the human M171 module model.
when_to_use_negative:
- Input is already a predicted flux or feature table rather than a gene-by-cell scRNA-seq
  count matrix — scFEA requires expression as input
- The goal is a coding-free web workflow or non-human/mouse organism modules — use
  the FLUXestimator web server (http://scFLUX.org/) instead, which serves curated
  networks for human, mouse and 15 other organisms
- Proteomics or metabolomics data are the input — flux estimation for other omics
  types is not yet available per the article's discussion
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3407
tools:
- name: scFEA
  role: Stand-alone graph neural network pipeline that trains the flux estimator on
    imputed scRNA-seq data and writes the flux and metabolite balance matrices
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web-server implementation of the same scFEA approach; the stand-alone tools
    for local use live in the scFEA GitHub repository
  repo: https://github.com/changwn/scFEA
- name: MAGIC
  role: Performs the single-cell expression imputation step before flux estimation;
    the imputation step was fixed in scFEA v1.1 to use MAGIC
  repo: https://www.krishnaswamylab.org/projects/magic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:38810c1ddf5c7943247bdae890e9ef5e005ed2e6ad6463d1c0029930c5f78112
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
yes_rate: 0.6875
n_claims: 16
---

# single-cell-metabolic-flux-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the stand-alone scFEA pipeline (the local implementation of FLUXestimator's scFEA method) to infer cell-wise metabolic flux and metabolite stress (balance/variation) matrices from a single-cell RNA-seq expression matrix using the M171 human metabolic module model and a graph neural network flux estimator.

## When to use

Use when a single-cell expression matrix (rows = genes, columns = cells, raw or normalized counts, e.g. Melissa_full.csv) needs cell-wise predicted metabolic flux and metabolite variation outputs from the human M171 module model.

## When NOT to use

- Input is already a predicted flux or feature table rather than a gene-by-cell scRNA-seq count matrix — scFEA requires expression as input
- The goal is a coding-free web workflow or non-human/mouse organism modules — use the FLUXestimator web server (http://scFLUX.org/) instead, which serves curated networks for human, mouse and 15 other organisms
- Proteomics or metabolomics data are the input — flux estimation for other omics types is not yet available per the article's discussion

## Inputs

- Single-cell expression matrix (rows = genes, columns = cells; raw or normalized counts, e.g. Melissa_full.csv in data/)
- scFEA model files from data_dir: moduleGene file (module_gene_m168.csv for human default), stoichiometry matrix (cmMat_171.csv), and cName compound-name file
- Optional flags: --sc_imputation {True,False} for MAGIC imputation, --output_flux_file and --output_balance_file for custom output names (v1.1.2)

## Outputs

- Predicted cell-wise metabolic flux matrix (one row per input cell, one column per module flux)
- Metabolite stress / balance (variation) matrix at single-cell resolution
- Both written to res_dir under names set by output_flux_file and output_balance_file

## How to apply

Clone the scFEA repository and install its Python3 dependencies (torch >= 0.4.1, numpy, pandas, matplotlib, magic >= 2.0.4 via magic-impute). Point --data_dir at the model files (default human module_gene_m168.csv, cmMat_171.csv stoichiometry matrix, and the cName compound-name file) and --input_dir at the single-cell input, which is a gene-by-cell matrix accepting raw or normalized counts (log is applied automatically if values exceed 30). Enable --sc_imputation True for 10x-style sparse data so MAGIC (fixed in v1.1) imputes the expression matrix before training. Train the scFEA graph neural network flux estimator on the imputed data against the module structure — running `python src/scFEA.py` with defaults or passing --test_file Melissa_full.csv — and write results to --res_dir. For datasets with more than 10,000 cells the general pipeline is slow without GPU; the README recommends (not-yet-released) sampling/fitting functions to train on a subset per cell type and then predict flux on all cells.

## Related tools

- **scFEA** (Stand-alone graph neural network pipeline that trains the flux estimator on imputed scRNA-seq data and writes the flux and metabolite balance matrices) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web-server implementation of the same scFEA approach; the stand-alone tools for local use live in the scFEA GitHub repository) — https://github.com/changwn/scFEA
- **MAGIC** (Performs the single-cell expression imputation step before flux estimation; the imputation step was fixed in scFEA v1.1 to use MAGIC) — https://www.krishnaswamylab.org/projects/magic

## Examples

```
python src/scFEA.py --input_dir data --res_dir output --test_file Melissa_full.csv
```

## Evaluation signals

- Both output files exist in res_dir and are readable tables with one row per input cell
- Flux matrix has one column per predicted flux variable for the M171 modules and balance matrix has one column per metabolite-balance variable
- All entries are finite numeric flux/balance estimates (no NaN from imputation or training failures)
- Torch, numpy, pandas, matplotlib and magic-impute versions meet the README requirements (torch >= 0.4.1, magic >= 2.0.4) and the run completes without the known cName NA bug (fixed in v1.1.2)
- Convergence of the flux balance and non-negative loss terms during training, as illustrated in supplementary Figure S6

## Limitations

- For datasets with more than 10,000 single cells the general pipeline is too time-consuming without GPU acceleration; sampling and fitting functions are listed as 'to be released soon'
- sc_imputation should be set True for 10x data; without imputation sparse data may train poorly
- Biochemical constraints such as redox, pH, or energy balance are not yet implemented in the flux estimation method
- Perturbation analysis and newly curated modules (e.g. methionine, copper ion) are only available in the stand-alone version pending web-server validation
- The mouse model requires swapping the moduleGene and stoichiometry files; default files are the human M168/M171 set

## Evidence
