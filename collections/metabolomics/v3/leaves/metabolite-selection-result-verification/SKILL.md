---
name: metabolite-selection-result-verification
description: Use when an iMAP SVM feature-selection run on the colorectal cancer serum
  dataset (113 metabolites, 234 samples in CRC/polyp/healthy groups) must be validated
  against the published 10-metabolite selection before downstream use.
when_to_use_negative:
- The input is raw spectral data (mzML/NetCDF) rather than a processed metabolite-by-sample
  data matrix — iMAP does not implement raw data preprocessing or peak picking/annotation
  modules
- The feature-selection task uses RF or Boruta rather than SVM, since the 10-metabolite
  signature with undecanedionate and indole-3-propionate is specific to the SVM module
  output
- The dataset is not the 234-sample CRC serum cohort (e.g. the 308-sample IBD stool
  dataset), where the Figure 1F anchor metabolites are not expected
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server providing the SVM feature-selection module whose output (selected_metabolites_svm.csv)
    is the object of verification
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Parent desktop platform; iMAP backend R scripts used for reproducible execution
    of the feature-selection step
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:36c01fe4ee4e2fe32a52a0e548004208e7ee324159332ef7b90c7b95239b73e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - Metaboanalyst
  - iMAP
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolite-selection-result-verification

## Summary

Verifies that iMAP's SVM feature-selection module produced the expected 10-metabolite signature from the 113-metabolite x 234-sample CRC serum dataset, checking output format, entry count, and inclusion of anchor metabolites against the published workflow results.

## When to use

Use when an iMAP SVM feature-selection run on the colorectal cancer serum dataset (113 metabolites, 234 samples in CRC/polyp/healthy groups) must be validated against the published 10-metabolite selection before downstream use.

## When NOT to use

- The input is raw spectral data (mzML/NetCDF) rather than a processed metabolite-by-sample data matrix — iMAP does not implement raw data preprocessing or peak picking/annotation modules
- The feature-selection task uses RF or Boruta rather than SVM, since the 10-metabolite signature with undecanedionate and indole-3-propionate is specific to the SVM module output
- The dataset is not the 234-sample CRC serum cohort (e.g. the 308-sample IBD stool dataset), where the Figure 1F anchor metabolites are not expected

## Inputs

- selected_metabolites_svm.csv (iMAP standard CSV result format) containing ranked metabolites with SVM importance scores
- Targeted serum metabolic profiling data matrix (113 metabolites x 234 samples)
- Sample information table with SampleID and group columns (66 CRC, 76 polyp, 92 healthy controls)

## Outputs

- Verification report confirming exactly 10 metabolite entries in selected_metabolites_svm.csv
- Confirmation that undecanedionate and indole-3-propionate are present in the selection, matching Figure 1F of the iMAP workflow results

## How to apply

After running iMAP's SVM feature-selection module (or its backend R scripts from github.com/IP4M/R-scripts-for-iMAP) on the targeted serum metabolic profiling matrix, rank metabolites by SVM importance score and retain the top 10. Write the result as selected_metabolites_svm.csv in iMAP's standard CSV result format, then validate the file: it must contain exactly 10 metabolite entries, and must include undecanedionate and indole-3-propionate, matching the selection shown in the iMAP workflow results interface (Figure 1F). If either anchor metabolite is missing or the entry count deviates, re-examine the input data matrix (SampleID and group table must align with the 66 CRC / 76 polyp / 92 healthy controls design) and rerun the module.

## Related tools

- **iMAP** (Web server providing the SVM feature-selection module whose output (selected_metabolites_svm.csv) is the object of verification) — https://imap.metaboprofile.cloud/
- **IP4M** (Parent desktop platform; iMAP backend R scripts used for reproducible execution of the feature-selection step) — https://github.com/IP4M

## Examples

```
Rscript svm_feature_selection.R --input serum_113metab_234samples.csv --sample-info sample_groups.csv --top-n 10 --output selected_metabolites_svm.csv
```

## Evaluation signals

- selected_metabolites_svm.csv contains exactly 10 metabolite entries
- Undecanedionate is present in the selected metabolite list
- Indole-3-propionate is present in the selected metabolite list
- Metabolites are ordered by SVM importance rank consistent with the module's scoring output
- Output format conforms to iMAP's standard CSV result format and matches the selection displayed in the iMAP workflow results interface (Figure 1F)

## Limitations

- The anchor-metabolite check (undecanedionate, indole-3-propionate) is specific to the CRC serum dataset; it is not a general-purpose validator for other datasets
- Backend R scripts are hosted externally (github.com/IP4M/R-scripts-for-iMAP) and no changelog is available, so script versions may drift from the published web-module behavior
- iMAP does not implement raw data preprocessing or peak picking/annotation, so inputs must already be a processed data matrix
- The article acknowledges iMAP has 'still many limitations' but does not enumerate them in the discussion

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] SVM feature selection on 234-sample CRC serum dataset: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] Backend R scripts location: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] README description of the module underpinning the feature-selection output: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [intro] iMAP does not handle raw preprocessing, so inputs must be processed matrices: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
