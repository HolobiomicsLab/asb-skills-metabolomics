---
name: univariate-metabolite-threshold-selection
description: Use when a preprocessed metabolomics peak-intensity/concentration matrix
  with a sample group table (SampleID, group) must be reduced to a subset of significantly
  changed metabolites via univariate thresholds (P, FDR, log2FC).
when_to_use_negative:
- Raw MS data still requiring peak picking and annotation — iMAP deliberately did
  not implement raw data preprocessing or peak picking/annotation modules
- The input matrix has not passed through data-processing (missing values, normalization,
  batch effect correcting, QC-based removal of unstable variables), so univariate
  statistics would be computed on unfiltered data
- The goal is phenotype prediction/model building rather than metabolite selection
  — use iMAP's automatic train-test separation, feature selection, and predictive
  model construction module instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server hosting the metabolites-selection layer (univariate threshold filtering
    plus Union/Intersection summary modules)
  repo: https://github.com/IP4M
- name: IP4M
  role: Desktop predecessor whose R-based metabolomics analysis workflows iMAP extends;
    same selection logic implemented in R
  repo: https://github.com/IP4M
- name: R-scripts-for-iMAP
  role: Backend R scripts implementing the univariate statistical analysis and threshold-based
    metabolite selection
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: ropls
  role: R package underlying multivariate PLS-DA/OPLS-DA VIP-based selection that
    complements univariate testing
- name: MetaboAnalystR
  role: R package used in iMAP's backend for statistical metabolomics analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:00af9e2c1fe74be554ac7f0cff89eb27e5ebf9e7d4d836fc651a357fd7ed7e69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hmisc
  - IP4M
  - MetaboAnalystR
  - R
  - R-scripts-for-iMAP
  - iMAP
  - ropls
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# univariate-metabolite-threshold-selection

## Summary

A metabolites-selection step from the iMAP framework (Zhou et al., 2021, Frontiers in Chemistry) in which statistical analysis on the processed peak-intensity matrix identifies/select significantly changed or phenotype-correlated metabolites using user-set P-value/FDR/|log2FC| thresholds, optionally combined with multivariate VIP-based selection via Union/Intersection summary modules.

## When to use

Use when a preprocessed metabolomics peak-intensity/concentration matrix with a sample group table (SampleID, group) must be reduced to a subset of significantly changed metabolites via univariate thresholds (P, FDR, log2FC).

## When NOT to use

- Raw MS data still requiring peak picking and annotation — iMAP deliberately did not implement raw data preprocessing or peak picking/annotation modules
- The input matrix has not passed through data-processing (missing values, normalization, batch effect correcting, QC-based removal of unstable variables), so univariate statistics would be computed on unfiltered data
- The goal is phenotype prediction/model building rather than metabolite selection — use iMAP's automatic train-test separation, feature selection, and predictive model construction module instead

## Inputs

- CSV preprocessed peak-intensity/concentration data matrix (output of iMAP data-processing layer, ending with statistics summary and PCA)
- Sample information table (CSV) with SampleID and group columns
- Optional variable (metabolite) information table (CSV)
- User-set thresholds for P-value, FDR, |log2FC|, and trend-across-groups criteria

## Outputs

- Selected-metabolites list (CSV artifact of the metabolites-selection layer)
- Univariate statistics table per metabolite (P, FDR, log2FC)
- Union/Intersection summary of univariate and VIP-based selections, feeding iMAP pathway modules (MSEA, MetPA)

## How to apply

In iMAP's Step 2 (metabolites-selection), take the data-processed matrix (missing values filled, normalized, batch-corrected, QC-unstable variables removed, transformed and scaled) and run univariate statistical analysis between sample groups. Apply user-set cutoffs on P-value, FDR, and absolute log2FC, optionally with a trends-across-groups filter for multi-group designs. Integrate univariate results with multivariate PLS-DA/OPLS-DA VIP-based selection, then reconcile the lists through iMAP's Union or Intersection summary modules. Selected metabolites satisfying the module thresholds become the input to Step 3 (data-interpretation, MSEA/MetPA).

## Related tools

- **iMAP** (Web server hosting the metabolites-selection layer (univariate threshold filtering plus Union/Intersection summary modules)) — https://github.com/IP4M
- **IP4M** (Desktop predecessor whose R-based metabolomics analysis workflows iMAP extends; same selection logic implemented in R) — https://github.com/IP4M
- **R-scripts-for-iMAP** (Backend R scripts implementing the univariate statistical analysis and threshold-based metabolite selection) — https://github.com/IP4M/R-scripts-for-iMAP
- **ropls** (R package underlying multivariate PLS-DA/OPLS-DA VIP-based selection that complements univariate testing)
- **MetaboAnalystR** (R package used in iMAP's backend for statistical metabolomics analysis)

## Examples

```
Rscript run_iMAP_metabolites_selection.R --input processed_matrix.csv --sample_info sample_table.csv --p 0.05 --fdr 0.05 --log2fc 1
```

## Evaluation signals

- The metabolites-selection layer emits a CSV artifact (selected-metabolites list) matching iMAP's native output format
- Every retained metabolite satisfies the user-set P/FDR/|log2FC| thresholds (and any trends-across-groups criterion)
- Union/Intersection summary correctly reconciles the univariate list with the PLS-DA/OPLS-DA VIP-based list
- The input matrix provably came from the data-processing layer (which terminated with a statistics summary and PCA) — statistics were not run on raw peak tables
- Downstream data-interpretation (MSEA/MetPA) runs successfully on the selected metabolite set, confirming the list is a valid pathway-analysis input

## Limitations

- Raw data preprocessing, peak picking, and annotation are not covered — iMAP assumes the matrix is already preprocessed ('Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security')
- Threshold choices (P, FDR, |log2FC|) are user-set, so the selected list is decision-sensitive and not uniquely determined
- iMAP has acknowledged limitations ('Although there are still many limitations, iMAP is an alternative tool') and no version number or changelog is stated

## Evidence

- [intro] Step 2: metabolites-selection. In this step, statistical analysis will be performed: "Step 2: metabolites-selection. In this step, statistical analysis will be performed"
- [methods] After data-processing, statistical analysis will be performed in various ways to select key metabolites ("metabolites-selection"): [evidence span withheld — anchor status: not_found]
- [intro] modules can be incorporated into thread-like or tree-like workflows by users: "iMAP also supports “customized workflows” mode. Modules belong to steps 1–3 can be incorporated into a thread-like or tree-like workflow"
- [intro] Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [readme] iMAP has more flexible workflows and more optional criteria for differential variables selection: "Compared with existing platforms, iMAP has more flexible workflows and more optional criteria for differential variables selection."
