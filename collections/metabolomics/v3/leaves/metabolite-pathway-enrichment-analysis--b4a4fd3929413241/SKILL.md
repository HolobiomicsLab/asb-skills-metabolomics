---
name: metabolite-pathway-enrichment-analysis
description: Use when a preprocessed peak-intensity/concentration matrix has already
  yielded a shortlist of significantly changed metabolites (via P/FDR/|log2FC| or
  PLS-DA/OPLS-DA VIP thresholds) and the goal is pathway-level biological interpretation
  via MSEA and MetPA.
when_to_use_negative:
- Input is raw spectral data — iMAP does not implement raw data preprocessing or peak
  picking and annotation, so spectra must already be preprocessed into a peak table
- No metabolites-selection result exists yet — pathway analysis must be run on metabolites
  that already passed the P/FDR/|log2FC| or VIP selection thresholds
- The goal is purely statistical/bivariate exploration (PCA, clustering) rather than
  biological interpretation of a metabolite list
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server hosting the three-step workflow whose data-interpretation layer
    provides the MSEA and MetPA pathway analysis modules
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Predecessor desktop platform for one-stop metabolomics data analysis from
    pre-processing to data-interpretation; iMAP backend R scripts derive from it
  repo: https://github.com/IP4M
- name: R-scripts-for-iMAP
  role: Backend R scripts implementing the iMAP modules including pathway analysis
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: MetaboAnalystR
  role: R package among the analysis tools used in the iMAP demonstration workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:92d0995ccd1eeb29cbf2f376ebf6bde3109a40697cb5d7204e9794fd9d9d8704
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hmisc
  - IMPaLA
  - IP4M
  - MPEA
  - MetPA
  - MetaboAnalystR
  - R
  - R-scripts-for-iMAP
  - iMAP
  - lilikoi
  - ropls
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolite-pathway-enrichment-analysis

## Summary

The data-interpretation step of the iMAP three-step metabolomics analysis framework (data-processing -> metabolites-selection -> data-interpretation), in which metabolite set enrichment analysis (MSEA) and metabolic pathway analysis (MetPA) are applied to a curated list of selected metabolites to reach a biologically significant conclusion.

## When to use

Use when a preprocessed peak-intensity/concentration matrix has already yielded a shortlist of significantly changed metabolites (via P/FDR/|log2FC| or PLS-DA/OPLS-DA VIP thresholds) and the goal is pathway-level biological interpretation via MSEA and MetPA.

## When NOT to use

- Input is raw spectral data — iMAP does not implement raw data preprocessing or peak picking and annotation, so spectra must already be preprocessed into a peak table
- No metabolites-selection result exists yet — pathway analysis must be run on metabolites that already passed the P/FDR/|log2FC| or VIP selection thresholds
- The goal is purely statistical/bivariate exploration (PCA, clustering) rather than biological interpretation of a metabolite list

## Inputs

- Preprocessed peak-intensity/concentration data matrix as CSV
- Sample information table (SampleID, group) as CSV
- Optional variable information table as CSV
- Selected-metabolites list produced by the metabolites-selection layer (univariate P/FDR/|log2FC| and/or PLS-DA/OPLS-DA VIP filtering)

## Outputs

- MSEA (metabolite set enrichment analysis) results as CSV
- MetPA (metabolic pathway analysis) results as CSV
- Processed data table from the data-processing layer as CSV
- Selected-metabolites list as CSV

## How to apply

Run the full iMAP three-step workflow, since pathway analysis (step 3, data-interpretation) must operate on metabolites selected in step 2 from data processed in step 1. Load the preprocessed peak-intensity/concentration data matrix as CSV together with a sample information table (SampleID and group) and an optional variable information table. In the data-processing layer, highlight missing values, remove samples with >90% zeros and variables with >40% zeros, fill missing values, normalize, correct batch effects, remove QC-unstable variables, then apply transformation and scaling, ending with a basic statistics summary and PCA. In the metabolites-selection layer, apply univariate analysis with user-set P/FDR/|log2FC| thresholds plus multivariate PLS-DA/OPLS-DA VIP-based selection, integrating the two results via the Union/Intersection summary modules. Finally run the data-interpretation layer on the selected metabolites using the pathway analysis modules (MSEA and MetPA), exporting each layer's intermediate artifact (processed data table, selected-metabolites list, pathway analysis results) as CSV to match iMAP's native output format.

## Related tools

- **iMAP** (Web server hosting the three-step workflow whose data-interpretation layer provides the MSEA and MetPA pathway analysis modules) — https://imap.metaboprofile.cloud/
- **IP4M** (Predecessor desktop platform for one-stop metabolomics data analysis from pre-processing to data-interpretation; iMAP backend R scripts derive from it) — https://github.com/IP4M
- **R-scripts-for-iMAP** (Backend R scripts implementing the iMAP modules including pathway analysis) — https://github.com/IP4M/R-scripts-for-iMAP
- **MetaboAnalystR** (R package among the analysis tools used in the iMAP demonstration workflow)

## Evaluation signals

- Each layer emits a CSV artifact (processed data table, selected-metabolites list, pathway analysis results) matching iMAP's native output format
- The data-processing layer terminates with a basic statistics summary and a PCA before any selection
- Metabolites entering data-interpretation satisfy the module selection thresholds (user-set P/FDR/|log2FC| or VIP-based criteria)
- MSEA and MetPA outputs are generated on the selected-metabolites list, not on the full unprocessed variable list

## Limitations

- Modules of raw data preprocessing and peak picking and annotation were not implemented in iMAP, so spectra must be preprocessed externally
- Although there are still many limitations, iMAP is an alternative tool — limitations acknowledged but not itemized in the article's discussion
- No version number stated for iMAP or IP4M, complicating reproducibility checks

## Evidence

- [intro] Step 3 is data-interpretation reaching a biologically significant conclusion: "Step 3: data-interpretation. The aim of this step is to reach a biologically significant conclusion"
- [methods] Data-processing precedes metabolites-selection in the iMAP workflow: [evidence span withheld — anchor status: not_found]
- [methods] Samples with >90% zeros and variables with >40% zeros were removed and normalized to counts per million: [evidence span withheld — anchor status: not_found]
- [readme] iMAP is freely available at the stated URL under MPL 2.0: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
