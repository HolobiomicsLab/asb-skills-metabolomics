---
name: r-backend-script-execution-for-metabolomics
description: Use when a metabolomics feature table (e.g., 113 metabolites x 234 CRC/polyp/healthy
  serum samples) must be processed through iMAP modules reproducibly via R scripts
  rather than the web interface, e.g., to rank metabolites by SVM importance.
when_to_use_negative:
- 'Raw spectra preprocessing: modules of raw data preprocessing and peak picking and
  annotation were not implemented in iMAP, so these scripts cannot replace XCMS-online/MZmine2-type
  raw-data processing'
- The input is not a pre-processed metabolite abundance/feature table but unprocessed
  instrument output
- A fully interactive exploratory analysis is desired rather than scripted, reproducible
  execution of a defined module
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server whose backend R scripts perform feature selection (SVM/RF/Boruta)
    and predictive model construction and validation on metabolomics data
  repo: https://github.com/IP4M
- name: IP4M
  role: Desktop predecessor platform; iMAP and its R scripts are the extended web-server
    version of the IP4M analysis pipeline
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:f40648f292353f93bcd054df07d195f792e6be4b47d9c719bdd61097629f5793
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - Metaboanalyst
  - iMAP
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# R backend script execution for metabolomics

## Summary

Runs iMAP's backend R scripts (hosted at github.com/IP4M/R-scripts-for-iMAP) directly to reproduce web-server modules such as SVM feature selection, enabling reproducible execution of metabolomics feature-selection and predictive-modeling pipelines outside the browser.

## When to use

Use when a metabolomics feature table (e.g., 113 metabolites x 234 CRC/polyp/healthy serum samples) must be processed through iMAP modules reproducibly via R scripts rather than the web interface, e.g., to rank metabolites by SVM importance.

## When NOT to use

- Raw spectra preprocessing: modules of raw data preprocessing and peak picking and annotation were not implemented in iMAP, so these scripts cannot replace XCMS-online/MZmine2-type raw-data processing
- The input is not a pre-processed metabolite abundance/feature table but unprocessed instrument output
- A fully interactive exploratory analysis is desired rather than scripted, reproducible execution of a defined module

## Inputs

- Targeted serum metabolic profiling data matrix (metabolites x samples, e.g., 113 metabolites x 234 samples)
- Sample information table with SampleID and group columns (66 CRC, 76 polyp, 92 healthy controls)
- iMAP backend R scripts from github.com/IP4M/R-scripts-for-iMAP

## Outputs

- CSV result file in iMAP standard format (e.g., selected_metabolites_svm.csv) containing ranked selected metabolites (top 10 with SVM importance ranks)

## How to apply

Load the targeted serum metabolic profiling data matrix together with a sample information table (SampleID and group) and pass them to the iMAP R scripts checked out from github.com/IP4M/R-scripts-for-iMAP, the same implementation behind the iMAP web modules. For feature selection, invoke the SVM selection script (the web module also offers RF and Boruta as alternatives) on the three-group serum dataset (66 CRC, 76 polyp, 92 healthy controls), then rank metabolites by the SVM importance scores and retain the top-scoring metabolites. Write the selected metabolites with their importance ranks to a standard iMAP CSV result file such as selected_metabolites_svm.csv. Rationale: the article states the backend R scripts are provided specifically so the web module's computations can be reproduced; the pipeline also covers train-test set separation, feature selection, and predictive model construction and validation.

## Related tools

- **iMAP** (Web server whose backend R scripts perform feature selection (SVM/RF/Boruta) and predictive model construction and validation on metabolomics data) — https://github.com/IP4M
- **IP4M** (Desktop predecessor platform; iMAP and its R scripts are the extended web-server version of the IP4M analysis pipeline) — https://github.com/IP4M

## Examples

```
Rscript run_svm_feature_selection.R --input serum_metabolites_113x234.csv --sample_info sample_groups.csv --top 10 --output selected_metabolites_svm.csv
```

## Evaluation signals

- The output CSV contains exactly 10 metabolite entries for the SVM top-10 selection
- The selection output includes undecanedionate and indole-3-propionate, matching the iMAP workflow results interface (Figure 1F)
- Ranks are written with importance scores in iMAP's standard CSV result format
- Execution uses the scripts from github.com/IP4M/R-scripts-for-iMAP, not a re-implementation
- The three-group structure (66 CRC, 76 polyp, 92 healthy controls) of the 234 samples is preserved through train-test separation

## Limitations

- Modules of raw data preprocessing and peak picking and annotation were not implemented in iMAP, citing risk of data security
- The article acknowledges many limitations of iMAP but does not specify them
- No version number is stated for iMAP or IP4M, and no changelog is available for the associated repository

## Evidence

- [methods] The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] 234 clinical samples from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls): "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [intro] Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [readme] web server of IP4M: "## README: IP4M/R-scripts-for-iMAP
# R-scripts-for-iMAP
web server of IP4M"
