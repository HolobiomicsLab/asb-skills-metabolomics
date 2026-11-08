---
name: svr-based-intensity-normalization
description: Use when an extracted LC-MS metabolic feature table has already been
  filtered for noisy features (>20% MVs in QC samples, >50% MVs in study groups) and
  missing values imputed (KNN), and intensities still need drift correction before
  statistical analysis.
when_to_use_negative:
- Input is still raw LC-MS data (mzML etc.) that has not undergone peak picking into
  a feature table
- The feature table contains missing values that have not yet been imputed — run the
  massCleaner noisy-feature filter and KNN imputation first, since SVR normalization
  is the final data-cleaning step in this workflow
- The analysis goal is annotation or metabolite ID assignment (e.g. filtering to level
  2 annotations) rather than intensity normalization prior to statistical analysis
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: tidyMass
  role: Overall object-oriented R workflow framework in which the normalization step
    is embedded and traced
  repo: https://www.tidymass.org/
- name: masscleaner
  role: Applies the noisy-feature filter (>20% MV in QC, >50% MV in study groups),
    KNN imputation, and SVR-based normalization
  repo: https://gitlab.com/tidymass/tidymass
- name: massdataset
  role: Provides the mass_dataset uniform data structure that stores the processed
    feature table and processing parameters for traceability
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: R
  role: Language in which the cleaning and normalization pipeline is executed
  repo: https://github.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:28567f78fcde6fa1299c81693d0babaea2284b86df16389b3ae9a9598984d3de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - masscleaner
  - massdataset
  - tidyMass
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# svr-based-intensity-normalization

## Summary

A data-cleaning step from the tidyMass CRC case study in which the KNN-imputed metabolic feature table is normalized using support vector regression (SVR) via the massCleaner package, removing sample- and feature-level intensity drift.

## When to use

Use when an extracted LC-MS metabolic feature table has already been filtered for noisy features (>20% MVs in QC samples, >50% MVs in study groups) and missing values imputed (KNN), and intensities still need drift correction before statistical analysis.

## When NOT to use

- Input is still raw LC-MS data (mzML etc.) that has not undergone peak picking into a feature table
- The feature table contains missing values that have not yet been imputed — run the massCleaner noisy-feature filter and KNN imputation first, since SVR normalization is the final data-cleaning step in this workflow
- The analysis goal is annotation or metabolite ID assignment (e.g. filtering to level 2 annotations) rather than intensity normalization prior to statistical analysis

## Inputs

- Processed LC-MS feature table (features × samples) from the CRC case study, loaded as a mass_dataset object
- Sample metadata identifying QC samples and study groups (needed for the 20% QC MV / 50% group MV thresholds)

## Outputs

- SVR-normalized intensity table exported as cleaned_feature_table.csv
- Record of features removed by each missing-value threshold (>20% MV in QC, >50% MV in study groups)
- Normalization summary counts from massCleaner

## How to apply

Load the case-study processed feature table into a mass_dataset object using the massdataset package in R. First apply the massCleaner noisy-feature filter — remove any feature with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups — because SVR normalization requires a complete, denoised matrix. Impute the remaining missing values in the filtered table with the KNN method provided by massCleaner. Then run the SVR-based normalization step in massCleaner on the imputed intensity table; SVR regression of each feature's intensity against QC-injected sample order corrects intensity drift within the tidyMass uniform mass_dataset structure, which stores both the processed data and processing parameters so prior steps can be traced. Export the cleaned, normalized table as cleaned_feature_table.csv, recording the number of features removed by each MV threshold and the normalization summary counts.

## Related tools

- **tidyMass** (Overall object-oriented R workflow framework in which the normalization step is embedded and traced) — https://www.tidymass.org/
- **masscleaner** (Applies the noisy-feature filter (>20% MV in QC, >50% MV in study groups), KNN imputation, and SVR-based normalization) — https://gitlab.com/tidymass/tidymass
- **massdataset** (Provides the mass_dataset uniform data structure that stores the processed feature table and processing parameters for traceability) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **R** (Language in which the cleaning and normalization pipeline is executed) — https://github.com/tidymass/tidymass

## Evaluation signals

- No features in the cleaned_feature_table.csv exceed the missing-value thresholds: every retained feature has ≤20% MVs in QC samples and ≤50% MVs across all study groups
- Missing values are fully imputed before normalization (KNN step completed with no remaining NAs in the matrix passed to SVR normalization)
- Normalization summary counts from massCleaner are recorded, and QC-sample intensity drift is reduced after SVR correction
- The output mass_dataset object retains the processing parameters (filter thresholds, KNN imputation, SVR normalization) supporting reproducibility and tracing of prior analysis steps
- Re-running the workflow reproduces the same cleaned_feature_table.csv feature count as reported in the CRC case study

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though the mass_dataset class can be converted to other tools' data structures
- The thresholds and SVR normalization are specific to the CRC case-study design; datasets lacking QC samples cannot support the 20%-MV-in-QC filter criterion
- SVR normalization is only as reliable as the preceding steps — poor peak picking or insufficient imputation quality propagates into the normalized intensities

## Evidence

- [intro] Features with more than 20% missing values in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [abstract] Uniform data format stores processed data and parameters for traceability: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [readme] tidymass README citation of the source paper: [evidence span withheld — anchor status: external_doc]
- [intro] Docker image containing the case-study data, code, and analysis environment: [evidence span withheld — anchor status: not_found]
