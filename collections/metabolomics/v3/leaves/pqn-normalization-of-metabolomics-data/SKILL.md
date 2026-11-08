---
name: pqn-normalization-of-metabolomics-data
description: Use when a peak-aligned X matrix (peaks × samples) with biological/blank
  class labels still shows dilution-driven intensity variation across biological samples
  and must be normalized before KNN imputation and G-log transformation in the Galaxy-M
  DIMS pipeline.
when_to_use_negative:
- The input is raw unaligned mass-spectral data (transient/.dat or centroided peaks)
  rather than an aligned X matrix — peak picking, calibration, SIM-stitching and Align
  Samples must run first.
- The matrix has not yet been through Blank Filtering and Sample Filtering, since
  PQN sits mid-way in the documented Galaxy-M DIMS cleansing sequence and its input
  should be the filtered peak table.
- The data are LC-MS features already processed by the XCMS-Basic-Birmingham-Pipe
  rather than the DIMS workflow data considered here.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: Galaxy-M
  role: Provides the PQN normalization tool within the DIMS data-cleansing workflow,
    operating on the DSO produced by Create DSO
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Execution platform in which the Galaxy-M PQN step is orchestrated as part
    of the reproducible end-to-end workflow
  repo: https://github.com/galaxyproject/galaxy/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:ef6c7e545a4121a5fdd194a8b85ecffafb339d1f467c94bb944a6aa804b9693d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pqn-normalization-of-metabolomics-data

## Summary

Applies probabilistic quotient normalization (PQN) to a peak-aligned DIMS feature matrix inside the Galaxy-M data-cleansing workflow, correcting for sample dilution differences before missing-value imputation and G-log variance stabilization.

## When to use

Use when a peak-aligned X matrix (peaks × samples) with biological/blank class labels still shows dilution-driven intensity variation across biological samples and must be normalized before KNN imputation and G-log transformation in the Galaxy-M DIMS pipeline.

## When NOT to use

- The input is raw unaligned mass-spectral data (transient/.dat or centroided peaks) rather than an aligned X matrix — peak picking, calibration, SIM-stitching and Align Samples must run first.
- The matrix has not yet been through Blank Filtering and Sample Filtering, since PQN sits mid-way in the documented Galaxy-M DIMS cleansing sequence and its input should be the filtered peak table.
- The data are LC-MS features already processed by the XCMS-Basic-Birmingham-Pipe rather than the DIMS workflow data considered here.

## Inputs

- DSO XML containing the peak-aligned X data matrix with row and column label information
- Class labels describing whether each sample is biological or blank (assembled by the Create DSO Galaxy-M tool)
- Peak table already passed through Blank Filtering and Sample Filtering (x-out-of-n biological-sample threshold)

## Outputs

- DSO XML with a dilution-corrected (PQN-normalized) X matrix retaining row/column labels and class labels
- Normalized matrix ready for KNN missing-value imputation and QC-optimized G-log transformation

## How to apply

In the Galaxy-M DIMS case-study workflow, PQN is applied as step 4 of the cleansing sequence: after Create DSO assembles the aligned X data matrix with row and column labels and class labels (biological vs blank) into a DSO XML, and after Blank Filtering (removing peaks as strong in blanks as in biological samples) and Sample Filtering (removing peaks failing to appear in x-out-of-n biological samples, x user-chosen). The PQN step corrects for sample dilution across the biological samples of the DSO, producing a dilution-corrected X matrix that is then passed to K-nearest-neighbours (KNN) missing-value imputation and subsequently to the G-log transformation, whose variance-stabilization parameter is optimized on the QC samples alone. Execute the step as a Galaxy-M tool inside the Galaxy platform (pre-installed VM available via GigaDB), and validate that the output DSO XML still carries the X matrix, labels, and class labels and that the retained peak table reflects the upstream filtering steps.

## Related tools

- **Galaxy-M** (Provides the PQN normalization tool within the DIMS data-cleansing workflow, operating on the DSO produced by Create DSO) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Execution platform in which the Galaxy-M PQN step is orchestrated as part of the reproducible end-to-end workflow) — https://github.com/galaxyproject/galaxy/

## Evaluation signals

- Confirm the output DSO XML still contains the X matrix, row/column labels, and class labels (biological vs blank) with dimensions unchanged by normalization.
- Check that the retained peak table matches the upstream Blank Filtering and Sample Filtering behaviour, so PQN did not reintroduce blank-dominated peaks.
- Verify the normalized matrix feeds forward into KNN imputation and then G-log transformation, and that the G-log parameter was optimized using only the QC samples.
- Compare per-sample total signal/quotient behaviour before and after PQN to confirm dilution differences among biological samples were reduced.

## Limitations

- No univariate statistics tools are included in the current Galaxy-M implementation, so downstream assessment relies on PCA with statistical tests rather than per-feature tests.
- The PCA tool has no graphical output; separation statistics are used instead of visual interpretation.
- Installation of the full Galaxy-M environment (WINE 32-bit mode, MSFileReader, MATLAB MCR) is error-prone; the pre-installed VM is recommended, though the GigaDB accession was 'to follow' at publication time.

## Evidence

- [abstract] through data cleansing, e.g. missing value imputation, to preparation for statistical analysis: "The range of tools presented spans from processing of raw data, e.g. peak picking and alignment, through data cleansing, e.g. missing value imputation, to preparation for statistical analysis"
- [abstract] preparation for statistical analysis, e.g. normalization and scaling: "preparation for statistical analysis, e.g. normalization and scaling"
- [abstract] Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [readme] A virtual machine implementation has been created to support reuse by the community as well as to act as a snapshot of reproducibility for the work in the GigaScience publication.: "A virtual machine implementation has been created to support reuse by the community as well as to act as a snapshot of reproducibility for the work in the GigaScience publication."
