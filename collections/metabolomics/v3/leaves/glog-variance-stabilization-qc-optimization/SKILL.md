---
name: glog-variance-stabilization-qc-optimization
description: Use when a DIMS peak-intensity matrix has been blank/sample filtered,
  PQN-normalized and KNN-imputed, QC samples exist, and a variance-stabilizing transform
  is needed before PCA or other multivariate statistics.
when_to_use_negative:
- Input is still raw instrument data (e.g. FT-ICR transient .dat files) rather than
  an aligned peak matrix — use Sum Transients/Process Transients/Mass Calibration
  and SIM-stitching first
- No QC samples are present in the dataset, since the G-log parameters are optimized
  on QC samples alone
- LC-MS feature tables processed via XCMS are the target; G-log here is documented
  within the DIMS cleansing workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Galaxy-M
  role: Provides the Create DSO and G-log transformation tools that combine the X
    matrix into a DSO and optimize the variance-stabilization parameter on QC samples
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Execution platform hosting the Galaxy-M cleansing tools as reproducible workflow
    steps
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:d5777dad7df6b01474ba5159acb1187b5df8064252390cb2fc1cdf54bf7970c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# glog-variance-stabilization-qc-optimization

## Summary

Applies the Galaxy-M G-log transformation as the final variance-stabilizing step of the DIMS data-cleansing pipeline, optimizing its transformation parameter on QC samples alone rather than on biological samples.

## When to use

Use when a DIMS peak-intensity matrix has been blank/sample filtered, PQN-normalized and KNN-imputed, QC samples exist, and a variance-stabilizing transform is needed before PCA or other multivariate statistics.

## When NOT to use

- Input is still raw instrument data (e.g. FT-ICR transient .dat files) rather than an aligned peak matrix — use Sum Transients/Process Transients/Mass Calibration and SIM-stitching first
- No QC samples are present in the dataset, since the G-log parameters are optimized on QC samples alone
- LC-MS feature tables processed via XCMS are the target; G-log here is documented within the DIMS cleansing workflow

## Inputs

- Aligned DIMS X data matrix with row and column labels
- Class labels describing whether each sample is biological or blank
- Inclusion input file listing only the QC samples for G-log parameter optimization
- Peak table already subjected to Blank Filtering, Sample Filtering, PQN and KNN imputation

## Outputs

- DSO XML containing the G-log transformed X matrix, row/column labels and class labels
- Optimized G-log variance-stabilization parameter value fitted on QC samples

## How to apply

First combine the aligned X data matrix with row and column label information and class labels (biological vs blank) into a DSO XML using the Create DSO Galaxy-M tool. Before running G-log, supply an inclusion input file that sets only the QC samples to be included, because the G-log variance-stabilization parameters are optimized on the QC samples alone. Run G-log as the last cleansing step, after Blank Filtering (peaks as strong in blanks as in biological samples are removed), Sample Filtering (peaks absent from x-out-of-n biological samples), probabilistic quotient normalization, and KNN missing value imputation. Validate the resulting DSO XML contains the X matrix, labels and class labels, and that the retained peak table still reflects the documented filter behavior of each preceding step.

## Related tools

- **Galaxy-M** (Provides the Create DSO and G-log transformation tools that combine the X matrix into a DSO and optimize the variance-stabilization parameter on QC samples) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Execution platform hosting the Galaxy-M cleansing tools as reproducible workflow steps) — https://github.com/galaxyproject/galaxy

## Evaluation signals

- The output DSO XML contains the X matrix, row and column labels, and biological-vs-blank class labels
- The inclusion input file used for G-log listed only QC samples, confirming parameter optimization was QC-based
- G-log was applied after Blank Filtering, Sample Filtering, PQN and KNN imputation, in that order
- The retained peak table still reflects each preceding filter step's documented behavior (blanks-strong peaks removed; x-out-of-n presence enforced)
- Transformed intensities are suitable for the downstream PCA step performed with PLS-Toolbox

## Limitations

- No univariate statistics tools are included in the current implementation
- The workflow assumes QC samples exist; without them the QC-based G-log parameter optimization cannot be configured as described
- The paper's DIMS test data (MetaboLights MTBLS79) lacked transient files in the accession, so full reproduction depends on data obtained from the original authors

## Evidence

- [abstract] Create DSO combines X matrix and class labels: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [abstract] Blank Filtering behavior: "Blank Filtering: compares peaks in biological samples to those appearing in any ‘blank’ samples and removes any that appear to be as strong in the blanks"
- [abstract] Cleansing-to-statistics scope of the workflow: "through data cleansing, e.g. missing value imputation, to preparation for statistical analysis"
- [readme] Galaxy-M tool files available for reuse: "Included are the tool files (both original code and .xml wrappers) for Metabolomics analysis"
