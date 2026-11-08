---
name: lc-ms-feature-table-cleaning
description: Use when you have a processed LC-MS untargeted metabolomics feature table
  (features x samples) with missing values in QC and biological samples, and you need
  a cleaned, imputed, normalized intensity table before statistical analysis.
when_to_use_negative:
- Input is still raw LC-MS data (mzML/NetCDF) requiring peak picking and grouping
  — use upstream raw-data processing (e.g. xcms) before this cleaning step
- The feature table is already cleaned, imputed, and normalized — re-filtering would
  discard additional features and double-normalize intensities
- You are filtering metabolite annotations (e.g. keeping only level 2 annotations
  or removing redundant annotations by annotation score) rather than removing noisy
  features by missing values — that is a distinct annotation-cleaning step
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: tidyMass
  role: Umbrella R framework whose modular packages host the data-cleaning step of
    the LC-MS untargeted metabolomics workflow
  repo: https://www.tidymass.org/
- name: masscleaner
  role: Performs the noisy-feature MV filtering, KNN imputation, and SVR normalization
    on the feature table
- name: massdataset
  role: Provides the uniform mass_dataset object that wraps the feature table and
    processing parameters for traceability
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: R
  role: Language in which the cleaning workflow is implemented and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:08585045ad24ef6058cd942676ef3243070a53525926d073e51628d88797cad6
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# LC-MS feature table cleaning (noisy-feature removal, KNN imputation, SVR normalization)

## Summary

A data-cleaning procedure from the tidyMass CRC case study that removes noisy metabolic features using missing-value thresholds (20% MV in QC samples, 50% MV across study groups), then imputes remaining missing values with KNN and normalizes intensities with SVR, all within the massCleaner/massdataset ecosystem in R.

## When to use

Use when you have a processed LC-MS untargeted metabolomics feature table (features x samples) with missing values in QC and biological samples, and you need a cleaned, imputed, normalized intensity table before statistical analysis.

## When NOT to use

- Input is still raw LC-MS data (mzML/NetCDF) requiring peak picking and grouping — use upstream raw-data processing (e.g. xcms) before this cleaning step
- The feature table is already cleaned, imputed, and normalized — re-filtering would discard additional features and double-normalize intensities
- You are filtering metabolite annotations (e.g. keeping only level 2 annotations or removing redundant annotations by annotation score) rather than removing noisy features by missing values — that is a distinct annotation-cleaning step

## Inputs

- Processed LC-MS feature intensity table (features x samples) from the case-study raw-data processing step
- Sample metadata distinguishing QC samples from biological study groups
- mass_dataset object (or feature table convertible to mass_dataset via massdataset)

## Outputs

- cleaned_feature_table.csv with noisy features removed, missing values KNN-imputed, and intensities SVR-normalized
- Counts of features removed by each missing-value threshold (20% MV in QC, 50% MV in study groups)
- SVR normalization summary counts

## How to apply

Load the processed feature table into a mass_dataset object using the massdataset package in R, since mass_dataset is the kernel data structure of the object-oriented tidyMass workflow. Apply the masscleaner noisy-feature filter with two thresholds: remove any feature with more than 20% missing values (MV) in QC samples, or more than 50% MVs in all the study groups — this mirrors the CRC case-study definition of noisy features. Impute remaining missing values in the filtered table using the KNN method provided by masscleaner. Finally, normalize the imputed intensity table with SVR-based normalization in masscleaner, then export the cleaned table and record how many features were removed by each MV threshold along with the normalization summary counts so the step is traceable and reproducible.

## Related tools

- **tidyMass** (Umbrella R framework whose modular packages host the data-cleaning step of the LC-MS untargeted metabolomics workflow) — https://www.tidymass.org/
- **masscleaner** (Performs the noisy-feature MV filtering, KNN imputation, and SVR normalization on the feature table)
- **massdataset** (Provides the uniform mass_dataset object that wraps the feature table and processing parameters for traceability) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **R** (Language in which the cleaning workflow is implemented and executed)

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- No feature in the cleaned table exceeds 20% MV in QC samples or 50% MV across all study groups
- The imputed table contains no remaining missing values (KNN imputation completed on all features)
- Feature count of the cleaned table equals original count minus the sum of per-threshold removal counts, which are logged
- Intensity distributions are comparable across samples after SVR normalization (normalization summary counts recorded)
- The processing parameters and thresholds are retrievable from the mass_dataset object, satisfying tidyMass traceability

## Limitations

- The 20%/50% MV thresholds are case-study-specific choices from the CRC study; other datasets may require re-optimization of the cutoffs
- TidyMass does not wrap or support some widely used metabolomics tools, though mass_dataset can be converted to other tools' data structures
- Cleaning only operates on a processed feature table; it cannot recover features already lost in upstream raw-data processing

## Evidence

- [intro] Noisy-feature removal thresholds: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [intro] Data cleaning as workflow stage: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment"
- [abstract] mass_dataset kernel: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [readme] tidyMass installation: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
