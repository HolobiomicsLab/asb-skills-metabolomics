---
name: sample-class-mapping-for-qc-filters
description: Use when a feature table from LC-MS untargeted metabolomics must be curated
  with QC filters (RSD, D-ratio, blank subtraction, LOESS drift correction) and sample
  types are not yet encoded as classes.
when_to_use_negative:
- Input is raw LC-MS data before feature detection and correspondence (retention time
  alignment, peak picking, deisotoping, integration) — mapping applies to the extracted
  data matrix, not raw files
- The run sequence contains no pooled/intrastudy QC or blank injections, so no class-based
  filter (RSD, D-ratio, blank-based, LOESS) can be evaluated
- Feature annotation is the goal — the article notes TidyMS lacks feature annotation
  capability, and class mapping does not address it
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: Provides the Data Container that holds the feature matrix together with the
    sample mapping linking injections to pooled/intrastudy QC, blank, and study-sample
    classes
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Implementation language; the sample mapping is assembled with the pandas-backed
    tabular data matrix inside TidyMS
- name: scikit-learn
  role: Builds PCA models of the pooled QC samples to evaluate reduction of time-related
    systematic variance after class-based correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:ea510653b0eeaa286551ea75c4d00e9bb26979be83d78205877b3c2751d318ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  - statsmodels
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sample-class-mapping-for-qc-filters

## Summary

Constructs the sample-class mapping (pooled/intrastudy QC injections, blanks, and study samples) that TidyMS-based QC filtering and LOESS batch correction require, since each filter (RSD, D-ratio, blank-based, LOESS) is evaluated against a specific sample class.

## When to use

Use when a feature table from LC-MS untargeted metabolomics must be curated with QC filters (RSD, D-ratio, blank subtraction, LOESS drift correction) and sample types are not yet encoded as classes.

## When NOT to use

- Input is raw LC-MS data before feature detection and correspondence (retention time alignment, peak picking, deisotoping, integration) — mapping applies to the extracted data matrix, not raw files
- The run sequence contains no pooled/intrastudy QC or blank injections, so no class-based filter (RSD, D-ratio, blank-based, LOESS) can be evaluated
- Feature annotation is the goal — the article notes TidyMS lacks feature annotation capability, and class mapping does not address it

## Inputs

- Extracted feature table (data matrix of features x samples) from LC-MS feature detection and correspondence, loadable as a pandas-backed tabular matrix
- Sample metadata / injection order associating each run with pooled/intrastudy QC, solvent/zero-volume/process blank, or study sample (NIST candidate RM 8231, SRM 1950) classes

## Outputs

- TidyMS Data Container holding the feature matrix plus per-sample class labels usable by QC filter and LOESS-correction routines
- Class-indexed subsets (e.g., pooled QC injection intensities ordered by acquisition time) consumed by downstream RSD, D-ratio, and LOESS steps

## How to apply

Load the extracted feature matrix into a TidyMS Data Container together with a sample mapping that labels each injection as a pooled/intrastudy QC sample, solvent/zero-volume/process blank, or study sample, using the pandas-backed tabular data matrix. The mapping then drives class-specific operations: blank-based filtering sets peak areas to zero when a plasma-sample peak area is 10-fold or less than the maximum blank peak area; the RSD filter uses pooled/intrastudy QC injections and eliminates features with RSD larger than 20%; the D-ratio filter compares the MAD of pooled QC samples to the MAD of all study samples; and the LOESS batch correction is restricted to features present in pooled/intrastudy QC samples, as required by Supporting Text 1. Correctness of the mapping is the precondition for every downstream filter, since misassigned sample classes silently corrupt RSD, D-ratio, and drift estimates.

## Related tools

- **TidyMS** (Provides the Data Container that holds the feature matrix together with the sample mapping linking injections to pooled/intrastudy QC, blank, and study-sample classes) — https://github.com/griquelme/tidyms
- **Python** (Implementation language; the sample mapping is assembled with the pandas-backed tabular data matrix inside TidyMS)
- **scikit-learn** (Builds PCA models of the pooled QC samples to evaluate reduction of time-related systematic variance after class-based correction)

## Evaluation signals

- Every sample in the feature table is assigned exactly one class label (pooled/intrastudy QC, blank, or study sample) with no unmapped or doubly assigned injections
- Restricting batch correction to features present in pooled/intrastudy QC samples yields a feature subset consistent with the Supporting Text 1 condition
- RSD values computed from mapped pooled QC injections reproduce the 20% cutoff behavior (features with RSD > 20% eliminated)
- D-ratio computed as MAD of pooled QC samples relative to MAD of all study samples is finite and only computed after correct class assignment
- PCA of mapped pooled QC injections before and after correction shows reduced time-related signal drift (qualitative acceptance check mirroring the paper's comparison)

## Limitations

- The mapping only supports QC procedures (system suitability check, system conditioning, signal drift evaluation, data curation); TidyMS does not cover feature annotation
- No software version numbers are stated for TidyMS, Python, or scikit-learn, so reproducing the exact mapping behavior may require checking the tidyms-notebooks examples
- Filters requiring pooled/intrastudy QC classes fail or are undefined if the acquisition sequence lacks such injections

## Evidence

- [results] The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [results] all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [intro] Assessment of data quality within a study using different types of QC samples: [evidence span withheld — anchor status: not_found]
- [abstract] pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation: "pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation"
