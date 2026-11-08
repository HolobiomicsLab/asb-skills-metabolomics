---
name: cell-type-flux-differential-testing
description: Use when per-cell metabolic fluxes have been predicted from scRNA-seq
  (scFEA) and must be compared across reported cell-type labels (e.g. cancer, myeloid,
  T cells) to test which population has the highest glucose metabolic rate.
when_to_use_negative:
- Input is raw gene expression without a prior flux-estimation step — this skill tests
  predicted fluxes, not expression values
- Cells lack reliable cell-type labels, since the test requires assigning each cell
  its reported cell-type annotation before pairwise comparison
- Goal is reconstruction of context-specific (disease/tissue/cell-type) metabolic
  networks rather than comparing flux across existing cell types — this capability
  is not yet available in FLUXestimator
tools:
- name: scFEA
  role: Graph neural network flux estimator that produces the per-cell fluxes (output_flux_file,
    output_balance_file) fed into the cell-type differential testing
  repo: https://github.com/changwn/scFEA
- name: MAGIC
  role: Imputes dropout in the scRNA-seq expression matrix prior to flux estimation
    so downstream flux comparisons are computed on imputed values
  repo: https://www.krishnaswamylab.org/projects/magic
- name: FLUXestimator
  role: Webserver implementation of the scFEA pipeline providing curated human metabolic
    networks (including Glucose-Glutamine modules) for flux prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:080d99459c442cb1a022e3da1af4b462f16fb33d00a19b89ef0f3a11ed321b8c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cell-type-flux-differential-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A downstream statistical skill that compares predicted per-cell metabolic fluxes (from scFEA/FLUXestimator) across annotated cell types with pairwise Mann–Whitney U tests to rank cell-type metabolic activity, e.g. establishing that cancer cells in the tumor microenvironment have the highest glucose metabolic flux on the Glucose-Glutamine network.

## When to use

Use when per-cell metabolic fluxes have been predicted from scRNA-seq (scFEA) and must be compared across reported cell-type labels (e.g. cancer, myeloid, T cells) to test which population has the highest glucose metabolic rate.

## When NOT to use

- Input is raw gene expression without a prior flux-estimation step — this skill tests predicted fluxes, not expression values
- Cells lack reliable cell-type labels, since the test requires assigning each cell its reported cell-type annotation before pairwise comparison
- Goal is reconstruction of context-specific (disease/tissue/cell-type) metabolic networks rather than comparing flux across existing cell types — this capability is not yet available in FLUXestimator

## Inputs

- Single-cell RNA-seq count matrix (rows = genes, columns = cells; raw or normalized counts, e.g. CSV)
- Human metabolic module model files from data_dir (module_gene_m168.csv, cmMat_171.csv stoichiometry matrix, cName_c70_m168.csv compound names)
- Per-cell cell-type annotations (e.g. cancer cell, myeloid cell, T cell) from the source datasets

## Outputs

- output_flux_file: per-cell metabolic reaction flux matrix
- output_balance_file: per-cell metabolite balance/stress matrix
- Pairwise Mann–Whitney U test results comparing glucose metabolic flux distributions across cell types (P values at threshold P < 0.01)

## How to apply

First impute dropout in the scRNA-seq count matrix with MAGIC (run scFEA with --sc_imputation True for 10x data), then run the scFEA graph-neural-network flux estimator with the human model files (moduleGene_file module_gene_m168.csv, stoichiometry_matrix cmMat_171.csv, cName_c70_m168.csv) to write per-cell reaction fluxes to output_flux_file and metabolite balances to output_balance_file. Assign each cell its reported cell-type label (cancer cell, myeloid cell, T cell) from the source dataset annotations. For the glucose metabolic module (covering lactate production, TCA cycle, and nucleotide and serine biosynthesis on the Glucose-Glutamine network), compare flux-rate distributions pairwise across the three cell types with the Mann–Whitney U test at significance threshold P < 0.01. For datasets with more than 10,000 cells, use the sampling function to train a model on a subset per cell type and the fitting function to predict flux on all cells, since the general pipeline is too time-consuming without GPU acceleration.

## Related tools

- **scFEA** (Graph neural network flux estimator that produces the per-cell fluxes (output_flux_file, output_balance_file) fed into the cell-type differential testing) — https://github.com/changwn/scFEA
- **MAGIC** (Imputes dropout in the scRNA-seq expression matrix prior to flux estimation so downstream flux comparisons are computed on imputed values) — https://www.krishnaswamylab.org/projects/magic
- **FLUXestimator** (Webserver implementation of the scFEA pipeline providing curated human metabolic networks (including Glucose-Glutamine modules) for flux prediction)

## Examples

```
python src/scFEA.py --input_dir input_dir --data_dir data_dir --res_dir output --sc_imputation True
```

## Evaluation signals

- The regenerated ordering reproduces the reported result: cancer cells have the highest glucose metabolic flux rates, followed by myeloid cells, then T cells
- All pairwise Mann–Whitney comparisons across the three cell types are significant at P < 0.01
- output_flux_file and output_balance_file contain one row per input cell with non-negative predicted fluxes and metabolite balances
- Each cell's cell-type label matches the annotation reported in the source dataset before testing
- For >10,000-cell datasets, convergence of the flux balance loss and non-negative loss during training is checkable (goodness of fitting based on convergence curves)

## Limitations

- Datasets with more than 10,000 single cells require the sampling/fitting functions (or GPU acceleration); the general pipeline is too time-consuming
- The cName_c70_m168.csv compound-name file previously contained NA values (fixed in scFEA v1.1.2); older model files may break the run
- Fluxes are inferred from transcriptomes via a quadratic flux-balance loss, so biochemical constraints such as redox, pH, or energy balance are not enforced; proteomics/metabolomics inputs are not supported

## Evidence

- [full_text] Pairwise Mann-Whitney U testing of module flux across cell types: [evidence span withheld — anchor status: not_found]
- [results] Expected ordering of glucose flux by cell type: [evidence span withheld — anchor status: not_found]
- [readme] MAGIC imputation prior to flux estimation: "Fix bugs in single cell imputation step by using [Magic](https://www.krishnaswamylab.org/projects/magic)."
- [readme] scFEA outputs consumed by the differential test: "The output of scFEA includes two matrices, predicted metabolic flux and metabolites stress at single cell resolution."
- [methods] Sampling/fitting requirement for large datasets: "user should use sampling function to extract subset of cells for each cell type to train a model. Then use fitting function to predict metabolic flux on all cells."
