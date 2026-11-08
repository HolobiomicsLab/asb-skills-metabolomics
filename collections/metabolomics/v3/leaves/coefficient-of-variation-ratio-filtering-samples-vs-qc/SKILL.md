---
name: coefficient-of-variation-ratio-filtering-samples-vs-qc
description: Use when a feature abundance table from MS-DIAL (with a sample annotation
  file labeling QC vs biological samples) needs quality filtering based on the ratio
  of coefficient of variation across samples vs QCs.
when_to_use_negative:
- Input features have already passed CV-ratio filtering or the analysis goal is imputation/normalization
  (e.g. probabilistic quotient normalization) rather than reproducibility-based feature
  removal
- The dataset contains no pooled QC injections, so the CV(samples)/CV(QC) ratio cannot
  be computed
- The data are raw instrument files rather than the two tab-delimited margheRita inputs
  (feature data file plus sample annotation file)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: margheRita
  role: Provides the CV-ratio filtering function (samples vs QCs) applied to MS-DIAL
    feature tables, with a default threshold of 1
  repo: https://github.com/emosca-cnr/margheRita
- name: R
  role: Runtime environment for the margheRita package and its pre-processing workflow
- name: MS-Dial
  role: Generates the feature abundance table that is the input to the CV-ratio filter
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:bfb2db4fc1958d068743f41895a556f663a6d7906e0340c2ba7bfda5fdfd1359
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - MS-Dial
  - R
  - margheRita
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# coefficient-of-variation-ratio-filtering-samples-vs-qc

## Summary

A margheRita pre-processing filter for LC-MS/MS untargeted metabolomics that keeps only features whose CV ratio between no-QC biological samples and pooled QC samples exceeds a threshold (default 1), removing metabolites with unacceptable technical reproducibility.

## When to use

Use when a feature abundance table from MS-DIAL (with a sample annotation file labeling QC vs biological samples) needs quality filtering based on the ratio of coefficient of variation across samples vs QCs.

## When NOT to use

- Input features have already passed CV-ratio filtering or the analysis goal is imputation/normalization (e.g. probabilistic quotient normalization) rather than reproducibility-based feature removal
- The dataset contains no pooled QC injections, so the CV(samples)/CV(QC) ratio cannot be computed
- The data are raw instrument files rather than the two tab-delimited margheRita inputs (feature data file plus sample annotation file)

## Inputs

- Feature abundance table from MS-DIAL (feature identifiers, m/z, retention times, MS/MS spectra, abundances; tab-delimited)
- Sample annotation file (tab-delimited) with columns id, injection_order, batch, class, biological_rep, technical_rep, where 'class' distinguishes QC from biological samples

## Outputs

- Filtered feature table containing only metabolites with CV ratio above threshold (e.g. cv_filtered_features.tsv; 303/539 features kept on the Urine demo)
- Summary of kept/removed feature counts (e.g. cv_filter_summary.txt)

## How to apply

Load the MS-DIAL-derived feature table and the sample annotation file (columns: id, injection_order, batch, class, biological_rep, technical_rep) into R with margheRita utilities, using the 'class' column to distinguish QC from no-QC samples. Compute CV = standard deviation / mean for each metabolite separately across no-QC samples and across QC samples, then form the ratio CV(no-QC)/CV(QC). Apply margheRita's CV filtering function, keeping only features whose CV ratio is higher than the threshold (default 1), i.e. features whose biological variability is at least as large as their QC variability. The rationale, per the margheRita documentation, is that metabolite signals should not be more variable in repeated QC injections than in biological samples. Count retained vs removed features and export the filtered table. On the demonstration 'Urine' dataset, this filter with the default threshold keeps 303 of 539 metabolites.

## Related tools

- **margheRita** (Provides the CV-ratio filtering function (samples vs QCs) applied to MS-DIAL feature tables, with a default threshold of 1) — https://github.com/emosca-cnr/margheRita
- **R** (Runtime environment for the margheRita package and its pre-processing workflow)
- **MS-Dial** (Generates the feature abundance table that is the input to the CV-ratio filter)

## Examples

```
cv_kept <- margheRita::cv_filtering(mRList)  # default threshold 1; retain features with CV(no-QC)/CV(QC) > 1
```

## Evaluation signals

- On the margheRita 'Urine' demonstration dataset (Zenodo 10.5281/zenodo.11243781), the filter must retain exactly 303 of 539 metabolites with the default threshold of 1
- The number of features in the output table equals the number of input features whose CV(no-QC samples) / CV(QC samples) is strictly higher than the threshold
- No feature whose QC-sample CV exceeds its no-QC-sample CV (ratio <= 1 at default threshold) remains in the output
- A written summary must report kept and removed counts consistent with the filtered table dimensions

## Limitations

- The default threshold of 1 is conservative and dataset-dependent; other thresholds change the retained-feature count and must be reported
- Requires reliable QC/biological class labels in the annotation file; mislabeled samples distort the ratio
- It is one of several ordered pre-processing steps (m/z defect filtering, NA filtering, imputation) — applying it in the wrong order can change results, as shown by margheRita's ordered filtering example (e.g. 'Features without appropriate m/z values: 56')

## Evidence

- [intro] only features that have a CV ratio between no-QC samples and QC sample higher than a given threshold (by default 1) are kept: "only features that have a CV ratio between no-QC samples and QC sample higher than a given threshold (by default 1) are kept"
- [intro] such as filtering by mass defects, filtering by coefficient of variation (samples vs QCs) and probabilistic quotient normalization;: "such as filtering by mass defects, filtering by coefficient of variation (samples vs QCs) and probabilistic quotient normalization;"
- [readme] a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles: "a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles"
