---
name: sample-filtering-threshold-tuning
description: Use when an aligned X data matrix (peaks × samples) from DIMS or LC-MS
  processing contains sparse peaks, and low-prevalence features must be removed before
  normalization, imputation, and statistical analysis.
when_to_use_negative:
- The peaks to remove are those appearing in too few technical replicates — that is
  Replicate Filtering (x-out-of-n technical replicates), a different filter step
- The peaks to remove are contaminants as strong in blank samples as in biological
  samples — that is Blank Filtering, which precedes Sample Filtering in the workflow
- Input is raw mass spectral data (e.g. .dat transients or unaligned peak lists) rather
  than an aligned X matrix with class labels
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: Galaxy-M
  role: Provides the Sample Filtering tool (original code and .xml Galaxy wrapper)
    applied to the DSO within the data-cleansing workflow
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform in which the Sample Filtering step is executed as part of the Galaxy-M
    workflow
  repo: https://github.com/galaxyproject/galaxy/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:af6b9b8b39bce07925c0ce251f047be0b3114ad74bcf3eee77e9e5d220019880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sample-filtering-threshold-tuning

## Summary

Galaxy-M's Sample Filtering step removes peaks that fail to appear in at least x-out-of-n biological samples (x user-chosen, n = number of biological samples), reducing sparse features before normalization and G-log transformation in metabolomics data cleansing.

## When to use

Use when an aligned X data matrix (peaks × samples) from DIMS or LC-MS processing contains sparse peaks, and low-prevalence features must be removed before normalization, imputation, and statistical analysis.

## When NOT to use

- The peaks to remove are those appearing in too few technical replicates — that is Replicate Filtering (x-out-of-n technical replicates), a different filter step
- The peaks to remove are contaminants as strong in blank samples as in biological samples — that is Blank Filtering, which precedes Sample Filtering in the workflow
- Input is raw mass spectral data (e.g. .dat transients or unaligned peak lists) rather than an aligned X matrix with class labels

## Inputs

- Aligned X data matrix (peaks × samples) with row and column labels from peak picking and alignment
- Class labels marking each sample as biological or blank (combined into DSO XML via Create DSO)
- User-chosen threshold x (minimum number of biological samples in which a peak must appear)
- Optional QC sample inclusion file (used downstream for G-log optimization, not for this filter)

## Outputs

- Filtered peak table retaining only peaks present in at least x-out-of-n biological samples
- Updated DSO XML containing the filtered X matrix, row/column labels, and biological/blank class labels

## How to apply

First combine the aligned X data matrix with row and column label information and class labels describing whether each sample is biological or blank into a DSO XML using the Create DSO Galaxy-M tool, since filtering depends on the biological/blank designation. Run Blank Filtering before Sample Filtering so peaks as strong in blanks as in biological samples are removed first; then apply Sample Filtering with a user-chosen x so only peaks appearing in at least x of the n biological samples are retained. In the DIMS case study workflow, the order is Blank Filtering, Sample Filtering, probabilistic quotient normalization (PQN), K-nearest-neighbours imputation, then G-log transformation — so Sample Filtering must precede normalization and must not use QC samples (QC samples are reserved exclusively for optimizing the G-log variance-stabilization parameter via an inclusion input file). Validate that the retained peak table reflects the documented x-out-of-n filter behavior and that the DSO XML still contains the X matrix, labels, and class labels.

## Related tools

- **Galaxy-M** (Provides the Sample Filtering tool (original code and .xml Galaxy wrapper) applied to the DSO within the data-cleansing workflow) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform in which the Sample Filtering step is executed as part of the Galaxy-M workflow) — https://github.com/galaxyproject/galaxy/

## Evaluation signals

- Every peak in the output matrix appears in at least x of the n biological samples; peaks failing this are absent
- The DSO XML retains the X matrix, row and column labels, and the biological vs blank class labels after filtering
- Filtering was applied after Blank Filtering and before PQN normalization, matching the documented DIMS case-study order
- QC samples were excluded from the filtering population (they enter later only via the G-log inclusion input file)
- Row/column label correspondence between the input aligned matrix and the filtered peak table is preserved (no sample reordering or relabeling)

## Limitations

- The x threshold is chosen by the user with no automated optimization described; n is fixed as the number of biological samples
- Sample Filtering only counts presence in biological samples — QC samples are not part of the x-out-of-n criterion
- Galaxy-M tool files were tested against a specific Galaxy commit (c429777c93680dcee449fe410f5360afbe673758) and Python 2.7 / R 3.0.1 / MATLAB 2014a stack; reimplementation may require version adjustments

## Evidence

- [abstract] Sample Filtering removes peaks failing to appear in x-out-of-n samples: [evidence span withheld — anchor status: not_found]
- [abstract] Blank Filtering precedes and complements Sample Filtering: "Blank Filtering: compares peaks in biological samples to those appearing in any ‘blank’ samples and removes any that appear to be as strong in the blanks"
- [abstract] Create DSO supplies class labels needed by the filters: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [readme] Galaxy-M distributes the filter as tool files with Galaxy wrappers: "Included are the tool files (both original code and .xml wrappers) for Metabolomics analysis"
