---
name: plsda-oplsa-vip-feature-selection
description: Use when a preprocessed metabolomics peak-intensity/concentration matrix
  with sample group labels (SampleID + group) must be reduced to a list of key metabolites
  driving class separation before pathway-level interpretation.
when_to_use_negative:
- Input is still raw MS data or unprocessed peak tables — raw data preprocessing and
  peak picking/annotation are not implemented in iMAP
- The goal is unsupervised overview only — use the PCA/statistics summary of the data-processing
  layer instead of supervised PLS-DA/OPLS-DA
- Sample group labels are absent or unreliable, making supervised class-discrimination
  modeling inappropriate
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server whose metabolites-selection layer executes PLS-DA/OPLS-DA VIP-based
    selection and the Union/Intersection summary modules
  repo: https://github.com/IP4M
- name: IP4M
  role: Desktop predecessor platform providing the same metabolomics analysis modules
    iMAP extends
  repo: https://github.com/IP4M
- name: ropls
  role: R backend package for PLS-DA/OPLS-DA modeling and VIP computation in the iMAP
    metabolites-selection scripts
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: MetaboAnalystR
  role: R backend for the multivariate statistical analysis modules behind iMAP
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: randomForest
  role: Alternative multivariate feature-selection backend in iMAP's R scripts
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: Boruta
  role: Alternative feature-selection backend for the metabolites-selection layer
  repo: https://github.com/IP4M/R-scripts-for-iMAP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:59ee2948251df3d7e1bb7847130244e82600b5873c25cfed175b21f55416d23b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Boruta
  - Hmisc
  - IP4M
  - MetaboAnalystR
  - R
  - iMAP
  - randomForest
  - ropls
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# PLS-DA/OPLS-DA VIP-based metabolite feature selection

## Summary

The metabolites-selection step of the iMAP framework, in which multivariate supervised modeling (PLS-DA/OPLS-DA with VIP scoring, alongside univariate thresholds) is used to identify and select significantly changed or phenotype-correlated metabolites from a preprocessed peak-intensity data matrix.

## When to use

Use when a preprocessed metabolomics peak-intensity/concentration matrix with sample group labels (SampleID + group) must be reduced to a list of key metabolites driving class separation before pathway-level interpretation.

## When NOT to use

- Input is still raw MS data or unprocessed peak tables — raw data preprocessing and peak picking/annotation are not implemented in iMAP
- The goal is unsupervised overview only — use the PCA/statistics summary of the data-processing layer instead of supervised PLS-DA/OPLS-DA
- Sample group labels are absent or unreliable, making supervised class-discrimination modeling inappropriate

## Inputs

- Preprocessed peak-intensity/concentration data matrix (CSV, output of the data-processing layer)
- Sample information table with SampleID and group labels (CSV)
- Optional variable/metabolite information table (CSV)

## Outputs

- Selected-metabolites list with VIP scores and univariate statistics (P/FDR, log2FC), exported as CSV
- PLS-DA/OPLS-DA model summary (scores, loadings, R2/Q2) and publication-ready plots
- Union/Intersection module summary integrating univariate and multivariate selections

## How to apply

After iMAP's Step 1 data-processing layer (missing-value handling, normalization, batch effect correcting, removal of QC-unstable variables, transformation and scaling), run Step 2 metabolites-selection using the multivariate module: fit PLS-DA or OPLS-DA models (e.g., via ropls or MetaboAnalystR backends) on the processed matrix with group labels as the response, and rank variables by VIP score. Combine these multivariate results with univariate selection at user-set P/FDR and |log2FC| thresholds (plus trends-across-groups filtering), then integrate both strategies through iMAP's Union or Intersection summary modules to define the final selected-metabolites list. The rationale is that metabolites-selection is a distinct layer in iMAP's three-step architecture — data-processing, metabolites-selection, data-interpretation — and its output feeds directly into MSEA/MetPA pathway analysis. Optionally, the new iMAP module's train-test set separation, feature selection, and predictive model construction and validation pipeline can be used to verify that selected features generalize.

## Related tools

- **iMAP** (Web server whose metabolites-selection layer executes PLS-DA/OPLS-DA VIP-based selection and the Union/Intersection summary modules) — https://github.com/IP4M
- **IP4M** (Desktop predecessor platform providing the same metabolomics analysis modules iMAP extends) — https://github.com/IP4M
- **ropls** (R backend package for PLS-DA/OPLS-DA modeling and VIP computation in the iMAP metabolites-selection scripts) — https://github.com/IP4M/R-scripts-for-iMAP
- **MetaboAnalystR** (R backend for the multivariate statistical analysis modules behind iMAP) — https://github.com/IP4M/R-scripts-for-iMAP
- **randomForest** (Alternative multivariate feature-selection backend in iMAP's R scripts) — https://github.com/IP4M/R-scripts-for-iMAP
- **Boruta** (Alternative feature-selection backend for the metabolites-selection layer) — https://github.com/IP4M/R-scripts-for-iMAP

## Examples

```
library(ropls); plsda <- opls(X, y, predI = 1, printL = FALSE); vip <- plsda@vipVn; selected <- names(vip)[vip >= 1]
```

## Evaluation signals

- The multivariate module must be run only on data that has passed through the data-processing layer (missing-value filtering, normalization, QC-unstable variable removal, transformation and scaling)
- Each selected metabolite must satisfy the user-set P/FDR and |log2FC| thresholds or the PLS-DA/OPLS-DA VIP criterion applied in the module
- The selected-metabolites list must be emitted as a CSV artifact in iMAP's native output format before entering the data-interpretation (MSEA/MetPA) layer
- PLS-DA/OPLS-DA models should show class separation consistent with the group labels in the sample information table (e.g., CRC vs polyp vs healthy control in the 234-sample colorectal cancer demonstration dataset)
- The Union and Intersection summary modules must return a defined, reproducible metabolite set given the same thresholds and model seed

## Limitations

- Modules of raw data preprocessing and peak picking and annotation were not implemented in iMAP, so feature selection requires an already preprocessed matrix
- Selection thresholds are user-set, so the selected-metabolites list is not uniquely determined without explicit P/FDR, |log2FC|, or VIP cutoff choices
- As a supervised method, PLS-DA/OPLS-DA selection risks overfitting without validation; iMAP addresses this via the new train-test set separation, feature selection, and predictive model construction and validation module

## Evidence

- [intro] Step 2: metabolites-selection. In this step, statistical analysis will be performed: "Step 2: metabolites-selection. In this step, statistical analysis will be performed"
- [methods] After data-processing, statistical analysis will be performed in various ways to select key metabolites: [evidence span withheld — anchor status: not_found]
- [readme] iMAP has more flexible workflows and more optional criteria for differential variables selection: "iMAP has more flexible workflows and more optional criteria for differential variables selection."
