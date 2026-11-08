---
name: r-package-feature-selection-execution
description: Use when a metabolomics data matrix (samples × metabolites) with group
  labels needs automated feature selection — e.g., recovering a confirmed/rejected
  metabolite set like the 39-metabolite CRC serum signature — before building a predictive
  model.
when_to_use_negative:
- Raw spectra still needing preprocessing — iMAP explicitly did not implement raw
  data preprocessing or peak picking and annotation modules
- Input is already a reduced feature set; running Boruta on a pre-selected signature
  adds no discriminating information
- Data has not passed iMAP's data-processing module (missing value handling, normalization,
  batch effect correcting, transformation, and scaling)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server hosting the automatic pipeline for train-test set separation, feature
    selection, and predictive model construction and validation
  repo: https://github.com/IP4M
- name: IP4M
  role: Desktop precursor platform whose R scripts form the iMAP backend executing
    Boruta feature selection
  repo: https://github.com/IP4M
- name: Boruta
  role: R package used within the backend to classify metabolites as confirmed or
    rejected by comparing each real metabolite's importance against permuted shadow
    features
  repo: https://github.com/IP4M/R-scripts-for-iMAP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:1d71d2d84e1394683cea6bf195c56c8f6fa0d9186ce03494a0695d3394a3dff4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Boruta
  - IP4M
  - iMAP
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# r-package-feature-selection-execution

## Summary

Executes R-backend feature selection (e.g., the Boruta algorithm from iMAP's IP4M R-scripts) on a metabolomics intensity matrix to identify confirmed phenotype-discriminating metabolites, as part of iMAP's automatic predictive-model pipeline.

## When to use

Use when a metabolomics data matrix (samples × metabolites) with group labels needs automated feature selection — e.g., recovering a confirmed/rejected metabolite set like the 39-metabolite CRC serum signature — before building a predictive model.

## When NOT to use

- Raw spectra still needing preprocessing — iMAP explicitly did not implement raw data preprocessing or peak picking and annotation modules
- Input is already a reduced feature set; running Boruta on a pre-selected signature adds no discriminating information
- Data has not passed iMAP's data-processing module (missing value handling, normalization, batch effect correcting, transformation, and scaling)

## Inputs

- Metabolite intensity matrix (samples × metabolites), e.g., 234 clinical samples × 113 metabolites from the CRC targeted serum dataset (Zhu et al., 2014)
- Sample information table with SampleID and group labels (e.g., colorectal cancer / polyp / healthy control)

## Outputs

- boruta_selected_metabolites.csv — one metabolite name per row for all Boruta-confirmed metabolites (e.g., the reported 39-metabolite signature including adipate and 3-methyladipate/pimelate)
- Feature-selected matrix ready for predictive model construction and validation

## How to apply

Load the metabolite intensity matrix together with a sample information table containing SampleID and group labels (e.g., the CRC targeted serum dataset: 234 samples — 66 colorectal cancer patients, 76 polyp patients, 92 healthy controls — profiled over 113 metabolites). Run the Boruta algorithm (R Boruta package, part of the iMAP backend R scripts) using the three-group labels as the target: Boruta compares each real metabolite's importance against permuted shadow features and classifies metabolites as confirmed or rejected. Extract the confirmed metabolites and write them, one name per row, to boruta_selected_metabolites.csv. This step sits inside iMAP's new automatic pipeline that covers train-test set separation, feature selection, and predictive model construction and validation, so the selected feature set should be retained as the input for downstream model building.

## Related tools

- **iMAP** (Web server hosting the automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation) — https://github.com/IP4M
- **IP4M** (Desktop precursor platform whose R scripts form the iMAP backend executing Boruta feature selection) — https://github.com/IP4M
- **Boruta** (R package used within the backend to classify metabolites as confirmed or rejected by comparing each real metabolite's importance against permuted shadow features) — https://github.com/IP4M/R-scripts-for-iMAP

## Examples

```
Rscript boruta_feature_selection.R --input crc_serum_113metabolites.csv --sample-info sample_info.csv --group-label group --output boruta_selected_metabolites.csv
```

## Evaluation signals

- boruta_selected_metabolites.csv exists and is valid CSV with one metabolite name per row
- Every listed metabolite is drawn from the 113-metabolite input matrix (no invented features)
- Applied to the CRC serum dataset, the selection recovers the reported 39-metabolite set, which includes adipate and 3-methyladipate/pimelate
- All confirmed metabolites carry Boruta's 'confirmed' status relative to permuted shadow features, not merely tentative attributes
- The selected feature set is passed forward intact into iMAP's predictive model construction and validation step

## Limitations

- Backend R scripts must be fetched from github.com/IP4M/R-scripts-for-iMAP; no changelog is available for the repository
- No version number is stated for iMAP or IP4M, complicating reproducibility pinning
- Feature selection is supervised — it requires group labels; iMAP's pipeline assumes phenotype grouping is available
- The article acknowledges that 'there are still many limitations' of iMAP without enumerating them

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [methods] 234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls): "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [readme] web server of IP4M: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
- [methods] Samples with >90% zeros and variables with >40% zeros were removed: [evidence span withheld — anchor status: not_found]
