---
name: multi-group-serum-metabolomics-data-handling
description: Use when the input is a targeted serum metabolic profiling matrix with
  more than two phenotype groups plus a sample-information table (SampleID, group),
  and the goal is to feed iMAP's data-processing and metabolite-selection modules.
when_to_use_negative:
- Input is raw mass-spectrometry data requiring peak picking and annotation — iMAP
  explicitly did not implement raw data preprocessing or peak picking/annotation modules
- Data has already passed missing-value handling, normalization, and scaling and only
  feature selection is needed
- The task is correlation-based network construction rather than feature selection
  / predictive modeling
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server providing the data-processing, metabolites-selection, and predictive-modeling
    modules applied to the multi-group serum dataset
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Desktop predecessor platform; its R-script backend executes the iMAP modules
    for reproducible feature selection
  repo: https://github.com/IP4M
- name: Metaboanalyst
  role: Referenced open-source metabolomics analysis platform used as comparison context
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:64545a1cd6c71f857f4afe8b171b91f0cbf8862fa784f44034790f9d72fb77b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - Metaboanalyst
  - iMAP
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-group-serum-metabolomics-data-handling

## Summary

Handles multi-group targeted serum metabolomics data matrices (e.g. 113 metabolites x 234 samples across CRC, polyp, and healthy-control groups) within iMAP/IP4M, preparing them for downstream feature selection and predictive modeling with SVM, RF, or Boruta.

## When to use

Use when the input is a targeted serum metabolic profiling matrix with more than two phenotype groups plus a sample-information table (SampleID, group), and the goal is to feed iMAP's data-processing and metabolite-selection modules.

## When NOT to use

- Input is raw mass-spectrometry data requiring peak picking and annotation — iMAP explicitly did not implement raw data preprocessing or peak picking/annotation modules
- Data has already passed missing-value handling, normalization, and scaling and only feature selection is needed
- The task is correlation-based network construction rather than feature selection / predictive modeling

## Inputs

- Targeted serum metabolic profiling data matrix (113 metabolites x 234 samples in three groups: 66 CRC, 76 polyp, 92 healthy controls)
- Sample information table with SampleID and group columns (CSV)

## Outputs

- Processed, normalized metabolomics data matrix ready for metabolites-selection
- selected_metabolites_svm.csv in iMAP's standard CSV result format (10 metabolites with importance ranks)
- Trained and validated predictive model from the train-test separation / feature-selection / model-construction pipeline

## How to apply

Load the metabolite abundance matrix (rows = metabolites, columns = samples) together with the sample information table containing SampleID and group labels (e.g. 66 CRC, 76 polyp, 92 healthy controls). First pass the data through iMAP's data-processing module, which highlights missing values, removes samples/variables exceeding missing-value thresholds, fills missing values, normalizes, corrects batch effects, removes unstable variables using quality-control samples, then transforms and scales variables; the article's concrete defaults removed samples with >90% zeros and variables with >40% zeros and normalized each sample to counts per million. After processing, apply the metabolites-selection step (statistical analysis) or the predictive-modeling module, which supports SVM (with RF and Boruta as alternatives) implemented by backend R scripts hosted at github.com/IP4M/R-scripts-for-iMAP. Rank metabolites by importance score and retain the top-ranked set (e.g. the top 10 metabolites) as the module's selection output in iMAP's standard CSV result format.

## Related tools

- **iMAP** (Web server providing the data-processing, metabolites-selection, and predictive-modeling modules applied to the multi-group serum dataset) — https://imap.metaboprofile.cloud/
- **IP4M** (Desktop predecessor platform; its R-script backend executes the iMAP modules for reproducible feature selection) — https://github.com/IP4M
- **Metaboanalyst** (Referenced open-source metabolomics analysis platform used as comparison context)

## Examples

```
Rscript run_imap_svm.R --data serum_metabolites_113x234.csv --sample_info sample_groups.csv --top 10 --out selected_metabolites_svm.csv
```

## Evaluation signals

- Output file selected_metabolites_svm.csv contains exactly 10 metabolite entries
- Selection includes undecanedionate and indole-3-propionate, matching the iMAP workflow results interface (Figure 1F)
- Preprocessing removed samples with >90% zeros and variables with >40% zeros, with per-sample normalization to counts per million
- Sample counts per group remain consistent with the input table (66 CRC, 76 polyp, 92 healthy controls)
- Results are reproducible via the backend R scripts at github.com/IP4M/R-scripts-for-iMAP

## Limitations

- Modules for raw data preprocessing and peak picking and annotation were not implemented in iMAP, citing data-security risk
- iMAP acknowledges that many limitations remain and positions itself only as an alternative tool
- No version number is stated for iMAP or IP4M, and no changelog exists for the associated repository

## Evidence

- [abstract] abstract: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] methods: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [supplementary] supplementary: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [intro] intro: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
