---
name: knn-missing-value-imputation-peak-table
description: Use when a DIMS/LC-MS peak table (X matrix of sample × m/z intensities)
  still contains missing values after Blank Filtering, Sample Filtering, Replicate
  Filtering and probabilistic quotient normalization, and a complete matrix is required
  before G-log transformation and PCA.
when_to_use_negative:
- Input is already a complete feature/peak matrix with no missing values (e.g. after
  Replicate Filtering removed sparse peaks) — imputation would be a no-op or add noise
- Missing values are so pervasive that most rows lack data; KNN imputation relies
  on having informative neighbours among the retained peaks
- You are working on raw spectral data (e.g. transient .dat files or unaligned peak
  lists) rather than the post-alignment, post-normalization X matrix — the article
  positions imputation in the data-cleansing stage after peak picking and alignment
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Galaxy-M
  role: Provides the KNN missing-value imputation tool as part of the DIMS data-cleansing
    chain in the Galaxy workflow
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Execution platform in which the Galaxy-M KNN imputation tool is exposed and
    chained after PQN normalization
  repo: https://github.com/galaxyproject/galaxy
- name: PLS-Toolbox
  role: Downstream statistical toolkit (PCA tools, version 7.0.3 / 8.0.2) whose PCA
    consumes the imputed matrix
- name: XCMS
  role: Produces the aligned feature table (feature detection, retention time correction,
    alignment) whose missing values the KNN imputation step fills
  repo: https://metlin.scripps.edu/xcms/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:7f09a992263473d9b1a96fc09e7bb5536d4fea39939d69df0f488e829c1d02c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  - XCMS
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# knn-missing-value-imputation-peak-table

## Summary

A data-cleansing step in the Galaxy-M DIMS metabolomics workflow that fills missing peak-intensity values in the filtered X data matrix using K-nearest-neighbours (KNN) imputation on the filtered and normalized matrix, so that subsequent G-log transformation and PCA operate on a complete matrix.

## When to use

Use when a DIMS/LC-MS peak table (X matrix of sample × m/z intensities) still contains missing values after Blank Filtering, Sample Filtering, Replicate Filtering and probabilistic quotient normalization, and a complete matrix is required before G-log transformation and PCA.

## When NOT to use

- Input is already a complete feature/peak matrix with no missing values (e.g. after Replicate Filtering removed sparse peaks) — imputation would be a no-op or add noise
- Missing values are so pervasive that most rows lack data; KNN imputation relies on having informative neighbours among the retained peaks
- You are working on raw spectral data (e.g. transient .dat files or unaligned peak lists) rather than the post-alignment, post-normalization X matrix — the article positions imputation in the data-cleansing stage after peak picking and alignment

## Inputs

- Aligned X data matrix (sample × m/z peak intensities) from the DIMS or LC-MS processing pipeline
- DSO XML produced by the Galaxy-M Create DSO tool, containing X matrix, row/column labels and biological-vs-blank class labels
- Peak table that has passed Blank Filtering, Sample Filtering (x-out-of-n biological samples) and probabilistic quotient normalization (PQN)

## Outputs

- Completed (missing-value-free) X data matrix / DSO XML ready for G-log transformation and PCA
- Imputed peak-intensity table in which each retained m/z feature has a value for every biological sample

## How to apply

Within the Galaxy-M DIMS pipeline, apply KNN imputation as a dedicated Galaxy-M tool after the data-cleansing chain (Blank Filtering, Sample Filtering, and PQN normalization) but before G-log transformation, because G-log parameters are optimized on QC samples and a complete matrix is needed for stable variance-stabilization. The input to the KNN step is the DSO XML created by the Create DSO tool, which bundles the X data matrix with row/column labels and class labels distinguishing biological samples from blanks, so imputation can be restricted to the biological-sample rows. Missing values arise from peaks that survived filtering but were absent in some samples; KNN fills them using similarity to the nearest neighbouring samples in the filtered and normalized matrix. Verify afterwards that the output DSO/matrix has no remaining missing intensities and that the imputed values have not changed the retained-peak table structure.

## Related tools

- **Galaxy-M** (Provides the KNN missing-value imputation tool as part of the DIMS data-cleansing chain in the Galaxy workflow) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Execution platform in which the Galaxy-M KNN imputation tool is exposed and chained after PQN normalization) — https://github.com/galaxyproject/galaxy
- **PLS-Toolbox** (Downstream statistical toolkit (PCA tools, version 7.0.3 / 8.0.2) whose PCA consumes the imputed matrix)
- **XCMS** (Produces the aligned feature table (feature detection, retention time correction, alignment) whose missing values the KNN imputation step fills) — https://metlin.scripps.edu/xcms/

## Examples

```
git clone https://github.com/Viant-Metabolomics/Galaxy-M && galaxyctl start; galaxy_run --tool "KNN Impute" --input aligned_dso.xml --output imputed_dso.xml
```

## Evaluation signals

- Output matrix contains no missing values (NaNs) across the biological-sample rows
- Row and column counts are unchanged relative to the input filtered/normalized matrix — no peaks or samples were dropped by imputation
- Class labels in the DSO XML (biological vs blank) are preserved after imputation, so blanks were not treated as biological samples
- Downstream G-log transformation of the imputed matrix completes without error, and PCA on the imputed data yields separable sample clusters with valid statistics-of-separation output

## Limitations

- Only a limited number of metabolomics-specific computational workflows were available before Galaxy-M; KNN imputation here is part of that specific workflow, not a general-purpose replacement for other imputation strategies
- The workflow recommends using the pre-installed virtual machine from GigaDB, whose GigaDB accession details were pending at the time of writing ('accession to follow'), complicating exact reproduction of the imputation step
- Imputation is applied to the filtered and PQN-normalized matrix; applying it earlier would distort the filter thresholds (e.g. x-out-of-n sample filtering) that rely on missingness patterns

## Evidence

- [abstract] missing value imputation: "The range of tools presented spans from processing of raw data, e.g. peak picking and alignment, through data cleansing, e.g. missing value imputation, to preparation for statistical analysis"
- [abstract] DSO structure with class labels: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [abstract] PQN normalization step: "preparation for statistical analysis, e.g. normalization and scaling"
- [abstract] PCA downstream of imputation: "principal components analysis (PCA) with associated statistical evaluation"
- [methods] KNN imputation naming: [evidence span withheld — anchor status: not_found]
- [abstract] VM reproducibility caveat: "Galaxy and data are all provided pre-installed in a virtual machine (VM) that can be downloaded from the GigaDB repository"
- [abstract] Filtering stage precedes imputation: [evidence span withheld — anchor status: not_found]
- [methods] Missing values arise post-filtering: [evidence span withheld — anchor status: not_found]
- [abstract] Reproducibility recommendation: "We recommend that Galaxy-M workflow files are included within the supplementary information of publications, enabling metabolomics studies to achieve greater reproducibility"
- [readme] Python version dependency: "[Python (version 2.7)](https://www.python.org/download/releases/2.7/)"
