---
name: blank-contamination-peak-area-filtering
description: Use when curating an extracted LC–MS feature table (e.g., from mzML feature
  detection) where blank-contaminated features must be zeroed before downstream RSD,
  prevalence, or D-ratio filters and statistical analysis.
when_to_use_negative:
- If the input is already a curated matrix where blank correction has been performed
  (e.g., downstream of the full TidyMS curation pipeline), applying this filter again
  would zero valid features.
- If the dataset lacks blank injections (solvent, zero-volume, or process), the blank
  maximum cannot be computed and this filter is not applicable.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TidyMS
  role: Provides the data curation pipeline and Data Container in which this blank-based
    filter is implemented as a step preceding the RSD, prevalence, and D-ratio filters.
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Programming language in which the blank-peak-area filter logic is executed
    within the TidyMS package.
  repo: https://github.com/griquelme/tidyms
- name: scikit-learn
  role: Used to build PCA score plots that monitor the effect of the blank filter
    and the overall curation pipeline on pooled/intrastudy QC samples.
- name: pyopenms
  role: Reads raw mzML files that feed feature detection upstream of the blank-peak-area
    filtering step.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:c857e5f6822370a214ddd23409773a1b79a2d7a7e4c0f042e7937cafbc91806d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - pyopenms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# blank-contamination-peak-area-filtering

## Summary

A data-curation filter from the TidyMS pipeline that zeroes a feature's peak area in plasma samples whenever the area is 10-fold or less than the maximum area of the same feature in solvent, zero-volume-injection, and process blanks, thereby removing blank-contaminated features.

## When to use

Use when curating an extracted LC–MS feature table (e.g., from mzML feature detection) where blank-contaminated features must be zeroed before downstream RSD, prevalence, or D-ratio filters and statistical analysis.

## When NOT to use

- If the input is already a curated matrix where blank correction has been performed (e.g., downstream of the full TidyMS curation pipeline), applying this filter again would zero valid features.
- If the dataset lacks blank injections (solvent, zero-volume, or process), the blank maximum cannot be computed and this filter is not applicable.

## Inputs

- Extracted feature table (data matrix) with peak areas per feature per sample, loaded into the TidyMS Data Container (Pandas-backed) from mzML-derived feature detection
- Blank peak-area matrix for solvent, zero-volume-injection, and process blanks

## Outputs

- A filtered data matrix where blank-contaminated peak areas (≤10× blank maximum) are set to zero
- Filter statistics indicating how many features (or sample-feature pairs) were zeroed by the blank correction

## How to apply

After extracting the feature table and applying the Rt (≥90 s) and pooled/intrastudy-QC-presence filters in the TidyMS Data Container, inspect each feature's peak area across all plasma study samples and compare it to the maximum peak area of the same feature observed in solvent blanks, zero-volume-injection blanks, and process blanks. If a feature's peak area in any plasma sample is 10-fold or less than that blank maximum, set its peak area to zero in that sample. This step precedes the RSD filter (≤20% in pooled/intrastudy QC), the 100% intraclass prevalence filter (area threshold 5), and the D-ratio filter (MAD ratio <10%). Rationale: features with areas comparable to blank levels reflect carryover or background contamination rather than true plasma signal and must be removed to retain only analytically robust features. The effect of the filter is monitored alongside the rest of the curation pipeline using PCA score plots built with scikit-learn.

## Related tools

- **TidyMS** (Provides the data curation pipeline and Data Container in which this blank-based filter is implemented as a step preceding the RSD, prevalence, and D-ratio filters.) — https://github.com/griquelme/tidyms
- **Python** (Programming language in which the blank-peak-area filter logic is executed within the TidyMS package.) — https://github.com/griquelme/tidyms
- **scikit-learn** (Used to build PCA score plots that monitor the effect of the blank filter and the overall curation pipeline on pooled/intrastudy QC samples.)
- **pyopenms** (Reads raw mzML files that feed feature detection upstream of the blank-peak-area filtering step.)

## Examples

```
python tidyms_curate.py --input raw_mzml_dir --blank_filter_ratio 10 --output curated_matrix.csv
```

## Evaluation signals

- No plasma sample should retain a peak area that is 10-fold or less than the maximum area of the same feature across solvent, zero-volume-injection, and process blanks (post-filter areas in such cases must be exactly 0).
- The number of zeroed sample-feature pairs recorded by the pipeline must match the count of comparisons meeting the 10-fold criterion, as verified in the filter statistics.
- Downstream PCA score plots of pooled/intrastudy QC samples should show the expected tightening relative to the pre-curation matrix, indicating blank-driven noise has been suppressed.
- Features surviving this filter must also survive the subsequent RSD (≤20%), 100% intraclass prevalence (area ≥5), and D-ratio (<10%) filters to appear in the final 665-feature matrix.
- Re-running the filter on a validation subset must reproduce identical zeroing decisions, confirming determinism given the same blank peak-area matrix.

## Limitations

- The filter depends on the presence and quality of solvent, zero-volume-injection, and process blanks; missing or poorly prepared blanks can misclassify contaminated or genuinely low-abundance features.
- Setting an area to zero rather than removing the feature means downstream prevalence and D-ratio filters operate on a matrix that contains structural zeros, which can affect feature-level statistics if not accounted for.
- The 10-fold cutoff is fixed in this pipeline; features with blank levels just below this ratio may still reflect background contamination.

## Evidence

- [results] Blank-based filter: peak area set to 0 if 10-fold or less than maximum blank peak area: [evidence span withheld — anchor status: not_found]
- [results] Filter discarding features not present in pooled/intrastudy QC samples as a condition for LOESS batch correction: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [results] RSD filter in pooled/intrastudy QC samples eliminating features with RSD larger than 20%: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] D-ratio filter (MAD of QC samples relative to MAD of all study samples) removing features with D-ratio of 10% or higher: "The prevalence filter was followed by the D-ratio filter [19], estimated as the MAD of pooled /intrastudy QC samples relative to the MAD of all study samples."
- [intro] Feature table produced by feature detection and correspondence: "These software packages perform feature detection and correspondence, and provide an extracted data matrix as output for subsequent analysis."
