---
name: metabolomics-feature-filtering-and-imputation
description: Use when you have a raw feature table from LC-MS or DIMS peak detection/alignment
  (e.g. XCMS output) that must be cleaned of low-quality, blank-derived or non-reproducible
  peaks and made statistics-ready via imputation and normalization.
when_to_use_negative:
- Input is already a cleaned, normalized feature matrix or a prepared DSO with class
  labels — re-filtering would discard valid features.
- Raw spectral data has not yet been through feature detection, retention time correction
  and alignment (e.g. raw Thermo/NetCDF files prior to XCMS-Basic-Birmingham-Pipe).
- The analysis requires univariate statistics — Galaxy-M's current implementation
  includes no univariate statistics tools, only multivariate preparation such as PCA.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M
  role: Hosts the Replicate Filtering, Blank Filtering, Sample Filtering, missing
    value imputation, normalization and Create DSO tools as Galaxy workflow steps
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: XCMS
  role: Upstream producer of the feature list (feature detection, retention time correction,
    grouping/alignment) that the filtering and imputation steps operate on
  repo: https://metlin.scripps.edu/xcms/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:5260b45fc8ee857ce0b038dbbcc04cdec0099e828d92cd0e73d99130186eb7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy-M
  - PLS-Toolbox
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-feature-filtering-and-imputation

## Summary

A data-cleansing stage of the Galaxy-M metabolomics pipeline in which the feature list produced by XCMS-based peak picking is subjected to Replicate Filtering, Blank Filtering and Sample Filtering, followed by missing value imputation and normalization to prepare the data matrix for statistics.

## When to use

Use when you have a raw feature table from LC-MS or DIMS peak detection/alignment (e.g. XCMS output) that must be cleaned of low-quality, blank-derived or non-reproducible peaks and made statistics-ready via imputation and normalization.

## When NOT to use

- Input is already a cleaned, normalized feature matrix or a prepared DSO with class labels — re-filtering would discard valid features.
- Raw spectral data has not yet been through feature detection, retention time correction and alignment (e.g. raw Thermo/NetCDF files prior to XCMS-Basic-Birmingham-Pipe).
- The analysis requires univariate statistics — Galaxy-M's current implementation includes no univariate statistics tools, only multivariate preparation such as PCA.

## Inputs

- Aligned spectral feature table output from XCMS-Basic-Birmingham-Pipe (feature detection, retention time correction, grouping/alignment)
- Class label information distinguishing biological, blank and QC samples
- CSV instruction files from ~/GalaxyM-TestData used by the filtering/imputation workflow steps

## Outputs

- Filtered and imputed X data matrix ready for statistical analysis
- DSO object created by the Create DSO tool combining the X matrix with row/column labels and class labels

## How to apply

Run the Galaxy-M cleansing tools on the aligned feature list from XCMS-Basic-Birmingham-Pipe: first apply Replicate Filtering to remove peaks that fail to appear in at least x-out-of-n technical replicates, then Blank Filtering to remove peaks that appear as strong in 'blank' samples as in biological samples, and Sample Filtering to remove peaks failing to appear in x-out-of-n biological samples (x chosen by the user, n = number of biological samples). After filtering, perform missing value imputation on the feature matrix and normalize it so the data are ready for statistical analysis. Combine the cleaned X data matrix with row/column labels and class labels distinguishing biological from blank/QC samples using the Create DSO tool, importing the CSV instruction files shipped in the GalaxyM-TestData folder where the stored workflows require them. The rationale is to prevent noise and carry-over features from inflating downstream multivariate models such as the PLS-Toolbox PCA and Scores Test.

## Related tools

- **Galaxy-M** (Hosts the Replicate Filtering, Blank Filtering, Sample Filtering, missing value imputation, normalization and Create DSO tools as Galaxy workflow steps) — https://github.com/Viant-Metabolomics/Galaxy-M
- **XCMS** (Upstream producer of the feature list (feature detection, retention time correction, grouping/alignment) that the filtering and imputation steps operate on) — https://metlin.scripps.edu/xcms/

## Evaluation signals

- Retained peaks appear in at least x-out-of-n technical replicates per the Replicate Filtering threshold; peaks failing this are absent from the output matrix.
- Peaks as strong in blank samples as in biological samples are removed by Blank Filtering.
- Sample Filtering removes peaks absent from more than (n - x) of the n biological samples, with x user-defined.
- The final matrix contains no missing values after imputation and is packaged as a DSO with row/column labels and biological vs blank/QC class labels.
- Downstream PCA on the cleaned DSO (published LC-MS case: MTBLS146) retained seven PCs describing at least 70% of the variance, and the Scores Test table matched Additional file 1: Table S4.

## Limitations

- The x thresholds for Replicate Filtering and Sample Filtering are user-chosen, so results depend on operator settings; the article does not prescribe fixed cutoffs.
- Blank Filtering requires genuine blank samples; if none are supplied, blank-derived features cannot be removed by this toolset.
- The Galaxy-M paper notes the pipeline stops at multivariate preparation and PCA — no univariate statistics tools are included in the current implementation.

## Evidence

- [abstract] Replicate Filtering: filters peaks that fail to appear in at least x-out-of-n technical replicates: "Replicate Filtering: filters peaks that fail to appear in at least x-out-of-n technical replicates"
- [abstract] Sample Filtering removes peaks failing to appear in x-out-of-n samples: [evidence span withheld — anchor status: not_found]
- [abstract] data cleansing via missing value imputation and preparation for statistics: "The range of tools presented spans from processing of raw data, e.g. peak picking and alignment, through data cleansing, e.g. missing value imputation, to preparation for statistical analysis"
- [abstract] Create DSO combines X matrix with labels and class labels: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [readme] CSV instruction files used by the workflows live in GalaxyM-TestData: "The pipelines/workflows also make use of CSV files for certain instructions and these can be found in the GalaxyM-TestData folder mentioned above."
