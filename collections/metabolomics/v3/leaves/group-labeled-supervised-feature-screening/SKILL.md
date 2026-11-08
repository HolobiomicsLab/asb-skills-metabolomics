---
name: group-labeled-supervised-feature-screening
description: Use when the input is a metabolite intensity matrix (samples × metabolites,
  e.g., 234 samples × 113 metabolites) with sample-group labels and the goal is to
  select the subset of metabolites predictive of the phenotype.
when_to_use_negative:
- Raw spectral data preprocessing or peak picking and annotation are needed — iMAP
  deliberately did not implement raw data preprocessing and peak picking and annotation
  modules considering the risk of data security
- The data have no phenotype group labels, so no supervised target exists for Boruta's
  shadow-feature importance comparison
- The goal is unsupervised statistical metabolites-selection (e.g., differential abundance
  testing) rather than feature selection against class labels in the predictive-model
  module
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server whose new module provides the automatic pipeline for train-test
    set separation, feature selection, and predictive model construction and validation
  repo: https://imap.metaboprofile.cloud/
- name: Boruta
  role: R feature-selection algorithm run on the metabolite intensity matrix with
    group labels as target, comparing real features against permuted shadow features
  repo: https://github.com/IP4M/R-scripts-for-iMAP
- name: IP4M
  role: Desktop predecessor platform from the same group whose backend R scripts form
    the basis of iMAP's analysis pipeline
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:2361093e1cf74813acf2156629bf1126931d020773d41581cfb6696ab6224ade
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Group-labeled supervised feature screening

## Summary

Applies supervised feature selection (e.g., the Boruta algorithm on iMAP's backend R scripts) to a metabolite intensity matrix using sample group labels (e.g., 66 colorectal cancer patients, 76 polyp patients, 92 healthy controls) to identify confirmed marker metabolites such as the 39-metabolite CRC serum signature including adipate and 3-methyladipate/pimelate.

## When to use

Use when the input is a metabolite intensity matrix (samples × metabolites, e.g., 234 samples × 113 metabolites) with sample-group labels and the goal is to select the subset of metabolites predictive of the phenotype.

## When NOT to use

- Raw spectral data preprocessing or peak picking and annotation are needed — iMAP deliberately did not implement raw data preprocessing and peak picking and annotation modules considering the risk of data security
- The data have no phenotype group labels, so no supervised target exists for Boruta's shadow-feature importance comparison
- The goal is unsupervised statistical metabolites-selection (e.g., differential abundance testing) rather than feature selection against class labels in the predictive-model module

## Inputs

- Metabolite intensity matrix (CSV) of 113 metabolites × 234 clinical serum samples (CRC targeted serum metabolomics, Zhu et al., 2014)
- Sample information table with SampleID and group labels (66 colorectal cancer patients, 76 polyp patients, 92 healthy controls)

## Outputs

- boruta_selected_metabolites.csv listing confirmed metabolites, one per row (target: the reported 39-metabolite signature including adipate and 3-methyladipate/pimelate)

## How to apply

Load the targeted serum metabolomics data matrix (234 clinical samples: 66 colorectal cancer patients, 76 polyp patients, 92 healthy controls; 113 metabolites) together with a sample information table containing SampleID and group labels. Within iMAP's new module — which automates train-test set separation, feature selection, and predictive model construction and validation — run the Boruta algorithm (R Boruta package, part of the backend R scripts at github.com/IP4M/R-scripts-for-iMAP) using the three-group labels as the target. Boruta compares each real metabolite's importance against permuted shadow features and classifies metabolites as confirmed or rejected. Extract the confirmed metabolites and write them one per row to boruta_selected_metabolites.csv. Rationale: Boruta's shadow-feature comparison gives an objective, permutation-based criterion for inclusion in the selected-metabolite set, as recovered from the CRC dataset of Zhu et al., 2014 (234 samples).

## Related tools

- **iMAP** (Web server whose new module provides the automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation) — https://imap.metaboprofile.cloud/
- **Boruta** (R feature-selection algorithm run on the metabolite intensity matrix with group labels as target, comparing real features against permuted shadow features) — https://github.com/IP4M/R-scripts-for-iMAP
- **IP4M** (Desktop predecessor platform from the same group whose backend R scripts form the basis of iMAP's analysis pipeline) — https://github.com/IP4M

## Examples

```
Rscript -e "library(Boruta); boruta <- Boruta(x = X, y = group); write.csv(data.frame(metabolite = getSelectedAttributes(boruta)), 'boruta_selected_metabolites.csv', row.names = FALSE)"
```

## Evaluation signals

- boruta_selected_metabolites.csv exists and is in CSV format
- Every listed metabolite is drawn from the 113-metabolite input matrix (no invented features)
- The output reproduces the reported selected-metabolite set of 39 metabolites including adipate and 3-methyladipate/pimelate on the Zhu et al., 2014 CRC serum dataset
- All 234 samples (66 CRC, 76 polyp, 92 healthy controls) are used with the three-group labels as the Boruta target
- Boruta's confirmed/rejected classification is based on comparison against permuted shadow features, not an arbitrary cutoff

## Limitations

- Reproduction of the exact 39-metabolite signature depends on the Boruta run reaching the same confirmed set; results can vary with random seed/shadow-feature permutations
- The article does not state version numbers for iMAP or IP4M, complicating exact reproducibility
- This skill only covers feature selection and predictive modeling; raw data preprocessing and peak picking and annotation were not implemented in iMAP

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] 234 clinical samples (Zhu et al., 2014) from three groups: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] backend R scripts are available at github repo: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [readme] README description of new feature-selection pipeline module: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [intro] raw data preprocessing not implemented: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
