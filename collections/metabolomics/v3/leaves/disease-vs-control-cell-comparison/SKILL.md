---
name: disease-vs-control-cell-comparison
description: Use when single-cell transcriptomics data contain both disease (e.g.
  AD) and healthy-control cells of the same type (e.g. neurons) and the goal is to
  identify which predicted metabolites distinguish the two groups.
when_to_use_negative:
- Cells are not matched within the same cell type — comparing mixed heterogeneous
  cell subpopulations across conditions averages out cell-type-specific metabolic
  differences
- The input is bulk transcriptomics with no cell-level disease/control grouping and
  the goal is cell-type-resolved metabolite comparisons
- You need context-specific reconstruction of disease/tissue/cell-type metabolic networks
  — scFEA/FLUXestimator does not yet support this
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scFEA
  role: Graph neural network model that predicts cell-wise metabolic flux and metabolite
    balance from the single-cell RNA-seq matrix; run with the M171 network and output_flux_file/output_balance_file
    parameters
  repo: https://github.com/changwn/scFEA
- name: Magic
  role: Performs the single-cell expression imputation step that scFEA applies before
    flux prediction (sc_imputation=True, recommended for 10x data)
  repo: https://www.krishnaswamylab.org/projects/magic
- name: FLUXestimator
  role: Webserver implementation of the scFEA pipeline used to predict cell-wise metabolic
    flux and metabolite variations from transcriptomics data
  repo: http://scFLUX.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:21b30e9ad5e7f221387da9422ce3b1bbc7426510eedf17da8e1fcbfd53b69b81
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
  - Magic
  - scFEA
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Disease-vs-control cell comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Contrast predicted metabolic flux and metabolite concentration (balance) changes between disease cells and healthy-control cells within a single cell type, using scFEA/FLUXestimator outputs to rank metabolites by the magnitude of the disease-vs-control distinction.

## When to use

Use when single-cell transcriptomics data contain both disease (e.g. AD) and healthy-control cells of the same type (e.g. neurons) and the goal is to identify which predicted metabolites distinguish the two groups.

## When NOT to use

- Cells are not matched within the same cell type — comparing mixed heterogeneous cell subpopulations across conditions averages out cell-type-specific metabolic differences
- The input is bulk transcriptomics with no cell-level disease/control grouping and the goal is cell-type-resolved metabolite comparisons
- You need context-specific reconstruction of disease/tissue/cell-type metabolic networks — scFEA/FLUXestimator does not yet support this

## Inputs

- Single-nucleus/single-cell RNA-seq count matrix (rows = genes, columns = cells; raw or normalized counts), e.g. ROSMAP snRNA-seq data
- Cell-type annotation per cell (e.g. neuron vs other types) to allow within-type subsetting
- Disease/control label per cell (e.g. AD vs healthy control)
- scFEA model files for the chosen network (module_gene_m171.csv, cmMat_171.csv stoichiometry matrix for M171)

## Outputs

- Predicted metabolic flux matrix at single-cell resolution (output_flux_file)
- Predicted metabolite concentration/balance (stress) matrix (output_balance_file)
- Ranked top-14 metabolite distinction table (e.g. metabolite_changes_ad_vs_control.csv)

## How to apply

Load the single-nucleus RNA-seq count matrix into the scFEA input directory (input_dir for single-cell files, data_dir for model files) and enable single-cell imputation with MAGIC, the imputation step scFEA uses (recommended True for 10x data). Run scFEA with a chosen network module — for the ROSMAP AD example, the M171 network (module_gene_m171.csv and cmMat_171.csv) — writing predictions via the output_flux_file and output_balance_file parameters. Then subset the predicted metabolite balance matrix to the cell type of interest (e.g. neuron cells), compute the aggregate balance change contrasting disease versus healthy-control cells, and rank metabolites by the magnitude of the distinction. Extract the top-ranked metabolites (14 in the ROSMAP case) and check directionality against reported biology: in ROSMAP neurons, glycolytic substrates and GABA are increased while glucose, nucleic acids, and branched-chain amino acids (valine, leucine, isoleucine) are decreased. Write the ranked table to a CSV (e.g. metabolite_changes_ad_vs_control.csv).

## Related tools

- **scFEA** (Graph neural network model that predicts cell-wise metabolic flux and metabolite balance from the single-cell RNA-seq matrix; run with the M171 network and output_flux_file/output_balance_file parameters) — https://github.com/changwn/scFEA
- **Magic** (Performs the single-cell expression imputation step that scFEA applies before flux prediction (sc_imputation=True, recommended for 10x data)) — https://www.krishnaswamylab.org/projects/magic
- **FLUXestimator** (Webserver implementation of the scFEA pipeline used to predict cell-wise metabolic flux and metabolite variations from transcriptomics data) — http://scFLUX.org/

## Examples

```
python src/scFEA.py --input_dir input --data_dir data --res_dir output --test_file rosmap_counts.csv --moduleGene_file module_gene_m171.csv --stoichiometry_matrix cmMat_171.csv --sc_imputation True
```

## Evaluation signals

- The ranked CSV contains exactly 14 metabolite rows ordered by AD-vs-control distinction magnitude
- Directionality matches the reported biology: glycolytic substrates and GABA increased; glucose, nucleic acids, and branched-chain amino acids (valine, leucine, isoleucine) decreased
- output_flux_file and output_balance_file both exist and are well-formed matrices with cells as rows/columns
- Comparisons were computed within a single cell type (neuron cells only), not on pooled disease-vs-control cells
- MAGIC imputation was applied (sc_imputation True) rather than raw counts being passed directly to the model

## Limitations

- Datasets with more than 10,000 single cells are too time-consuming for the general pipeline without GPU; scFEA recommends a sampling function (train on a subset per cell type) plus a fitting function to predict flux on all cells
- scFEA balances are predicted, not measured — validation relies on consistency with matched metabolomics data or reported findings
- Some features (perturbation analysis, methionine/copper ion modules) are only available in the stand-alone version pending validation
- Log transform is applied automatically when input values exceed 30; inputs may be raw or normalized counts

## Evidence

- [readme] Fix bugs in single cell imputation step by using [Magic]: "Fix bugs in single cell imputation step by using [Magic](https://www.krishnaswamylab.org/projects/magic)."
- [readme] Output parameters for flux and balance files: "Add parameter `output_flux_file` and `output_balance_file` which allows user define custom output file names for predicted flux and balance files."
