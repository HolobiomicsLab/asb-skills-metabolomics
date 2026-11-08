---
name: targeted-serum-metabolomics-matrix-handling
description: Use when the input is a targeted serum metabolomics intensity matrix
  (samples x metabolites, e.g. 234 clinical samples, 113 metabolites) plus a sample
  table with SampleID and group labels, and the goal is automated feature selection
  such as Boruta before predictive model building.
when_to_use_negative:
- Input is raw spectrometry data needing peak picking and annotation — iMAP explicitly
  does not implement raw data preprocessing or peak picking and annotation modules
- The metabolite matrix has not passed through iMAP's data-processing module (missing-value
  handling, normalization, transformation, scaling); run that first
- The dataset lacks phenotype/group labels, since Boruta requires group labels as
  the classification target
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: iMAP
  role: Web server whose new module provides the automatic pipeline for train-test
    set separation, feature selection (Boruta), and predictive model construction
    and validation applied to the CRC serum dataset
  repo: https://imap.metaboprofile.cloud/
- name: Boruta
  role: R feature-selection algorithm comparing each real metabolite's importance
    against permuted shadow features to classify metabolites as confirmed/rejected
- name: IP4M
  role: Desktop predecessor platform whose R backend scripts (github.com/IP4M/R-scripts-for-iMAP)
    power the iMAP feature-selection workflow
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:7ff2b54434e9647b678ea42c53910a9359e51ef773862c426a357de34fea889a
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# targeted-serum-metabolomics-matrix-handling

## Summary

Loads and prepares a targeted serum metabolomics intensity matrix (e.g., the Zhu et al. 2014 CRC dataset: 234 samples, 113 metabolites) together with its SampleID/group-label sample information table, and runs Boruta feature selection via the iMAP backend R scripts to recover a confirmed-metabolite set (39 metabolites including adipate and 3-methyladipate/pimelate).

## When to use

Use when the input is a targeted serum metabolomics intensity matrix (samples x metabolites, e.g. 234 clinical samples, 113 metabolites) plus a sample table with SampleID and group labels, and the goal is automated feature selection such as Boruta before predictive model building.

## When NOT to use

- Input is raw spectrometry data needing peak picking and annotation — iMAP explicitly does not implement raw data preprocessing or peak picking and annotation modules
- The metabolite matrix has not passed through iMAP's data-processing module (missing-value handling, normalization, transformation, scaling); run that first
- The dataset lacks phenotype/group labels, since Boruta requires group labels as the classification target

## Inputs

- Targeted serum metabolomics intensity matrix (samples x metabolites), e.g. 234 clinical samples x 113 metabolites from Zhu et al. 2014
- Sample information table containing SampleID and three-group labels (66 colorectal cancer, 76 polyp, 92 healthy control)

## Outputs

- boruta_selected_metabolites.csv listing Boruta-confirmed metabolite names, one per row (expected ~39 metabolites including adipate and 3-methyladipate/pimelate)

## How to apply

Load the metabolite intensity matrix (here 234 clinical samples: 66 colorectal cancer patients, 76 polyp patients, 92 healthy controls; 113 metabolites) alongside the sample information table keyed by SampleID with three-group labels. Run the Boruta algorithm (R Boruta package, part of the iMAP backend R scripts at github.com/IP4M/R-scripts-for-iMAP) on the intensity matrix using the three-group labels as the target, so each real metabolite's importance is compared against permuted shadow features and metabolites are classified as confirmed or rejected. Extract the confirmed metabolites and write their names, one per row, to boruta_selected_metabolites.csv. This step sits inside iMAP's new module whose automatic pipeline covers train-test set separation, feature selection, and predictive model construction and validation.

## Related tools

- **iMAP** (Web server whose new module provides the automatic pipeline for train-test set separation, feature selection (Boruta), and predictive model construction and validation applied to the CRC serum dataset) — https://imap.metaboprofile.cloud/
- **Boruta** (R feature-selection algorithm comparing each real metabolite's importance against permuted shadow features to classify metabolites as confirmed/rejected)
- **IP4M** (Desktop predecessor platform whose R backend scripts (github.com/IP4M/R-scripts-for-iMAP) power the iMAP feature-selection workflow) — https://github.com/IP4M

## Examples

```
Rscript boruta_feature_selection.R --input serum_metabolite_matrix.csv --samples sample_info.csv --group_labels CRC,Polyp,Control --output boruta_selected_metabolites.csv
```

## Evaluation signals

- boruta_selected_metabolites.csv exists and is a valid CSV file
- Every listed metabolite name is drawn from the 113-metabolite input matrix
- The selected set recovers the reported 39-metabolite signature, including adipate and 3-methyladipate/pimelate
- Each selected metabolite is a Boruta-confirmed feature (not tentatively rejected) relative to permuted shadow features on the 234-sample, three-group labels

## Limitations

- Reproducing the exact 39-metabolite set depends on Boruta's stochastic shadow-feature permutations; confirmed/rejected calls can vary between runs
- iMAP does not implement raw data preprocessing or peak picking and annotation, so the input must already be a processed intensity matrix
- The article acknowledges 'there are still many limitations' of iMAP without enumerating them in the discussion

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] 234 clinical samples CRC dataset: "234 clinical samples (Zhu et al., 2014) from three groups (66 colorectal cancer patients, 76 polyp patients, and 92 healthy controls)"
- [methods] backend R scripts location: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
- [intro] raw data preprocessing not implemented: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [readme] iMAP freely available web server: "iMAP is freely available for academic usage at https://imap.metaboprofile.cloud/ (License MPL 2.0)."
