---
name: missing-value-filtering-qc-thresholds
description: Use when a processed LC-MS feature table contains missing values and
  you must decide which features to drop before statistical analysis, following the
  tidyMass CRC case-study criteria.
when_to_use_negative:
- The input is already a cleaned feature table with no missing values, so filtering
  would remove nothing and imputation/normalization would be redundant.
- The task is raw data processing (peak picking and grouping) — this skill operates
  on an already-extracted feature table.
- The workflow requires a different missing-value policy (e.g., retaining high-MV
  features); the 20%/50% thresholds are the CRC case study's definition of noisy features
  and would remove data you intend to keep.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: masscleaner
  role: Performs the noisy-feature MV filtering (20% QC / 50% study-group thresholds),
    KNN imputation, and SVR normalization
- name: massdataset
  role: Loads the processed feature table into the mass_dataset object that massCleaner
    operates on
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidyMass
  role: R framework providing the massCleaner/massdataset packages and uniform data
    format used for this cleaning step
  repo: https://github.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:fb0cb2b33af930c2d79606b80fc5bbc7655155023fe2e2efc9895990995559d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - masscleaner
  - massdataset
  - tidyMass
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Missing-value filtering with QC thresholds

## Summary

Removes noisy metabolic features from an LC-MS feature table before statistical analysis using missing-value thresholds (>20% MV in QC samples, >50% MV in study groups), followed by KNN imputation and SVR normalization in the tidyMass/massCleaner pipeline.

## When to use

Use when a processed LC-MS feature table contains missing values and you must decide which features to drop before statistical analysis, following the tidyMass CRC case-study criteria.

## When NOT to use

- The input is already a cleaned feature table with no missing values, so filtering would remove nothing and imputation/normalization would be redundant.
- The task is raw data processing (peak picking and grouping) — this skill operates on an already-extracted feature table.
- The workflow requires a different missing-value policy (e.g., retaining high-MV features); the 20%/50% thresholds are the CRC case study's definition of noisy features and would remove data you intend to keep.

## Inputs

- Processed LC-MS feature/peak intensity table (from the CRC case study processing step)
- Sample-class metadata identifying QC samples and study groups
- mass_dataset object (tidyMass uniform data structure)

## Outputs

- cleaned_feature_table.csv — the filtered, KNN-imputed, SVR-normalized feature table
- Counts of features removed by the 20% QC-MV threshold and the 50% study-group-MV threshold
- Normalization summary counts from SVR normalization

## How to apply

Load the case-study processed feature table into a mass_dataset object using the massdataset package. Apply the massCleaner noisy-feature filter, removing any feature with more than 20% missing values (MV) in QC samples or more than 50% MVs across all study groups — these are the two thresholds the CRC case study defines for 'noisy features'. Record how many features are removed by each threshold separately so the filtering is traceable, consistent with tidyMass's uniform data format that stores processing parameters alongside data. After filtering, impute remaining missing values with the massCleaner KNN method, then apply SVR-based normalization to the imputed intensity table. The cleaned table is then ready for metabolite annotation and statistical analysis steps.

## Related tools

- **masscleaner** (Performs the noisy-feature MV filtering (20% QC / 50% study-group thresholds), KNN imputation, and SVR normalization)
- **massdataset** (Loads the processed feature table into the mass_dataset object that massCleaner operates on) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidyMass** (R framework providing the massCleaner/massdataset packages and uniform data format used for this cleaning step) — https://github.com/tidymass/tidymass

## Evaluation signals

- Features removed satisfy the stated rule: >20% MV in QC samples OR >50% MVs in all study groups — no feature outside these criteria is dropped.
- Separate counts are recorded for features removed by each MV threshold, enabling traceability of the filtering step.
- After KNN imputation, the cleaned feature table contains no remaining missing values.
- SVR normalization summary counts are reported, confirming the normalization step ran on the imputed table.
- The output is exported as cleaned_feature_table.csv and can be loaded back as a mass_dataset for downstream annotation/statistics.

## Limitations

- The 20%/50% thresholds are the CRC case study's choices; other studies may require different MV cutoffs.
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though mass_dataset can be converted to other tools' data structures.
- Filtering depends on correct QC-sample and study-group labels in the metadata; mislabeled classes will corrupt the MV statistics.

## Evidence

- [intro] Features with more than 20% missing values in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [abstract] tidyMass designed to store and manage processed metabolomics data and processing parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [readme] tidymass installation instructions from README: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
