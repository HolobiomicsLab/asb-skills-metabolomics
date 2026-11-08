---
name: knn-missing-value-imputation
description: Use when an extracted LC-MS feature table still contains missing values
  after noisy features (>20% MVs in QC samples or >50% MVs across study groups) have
  been removed, and a complete intensity matrix is needed for statistical analysis.
when_to_use_negative:
- Input is raw LC-MS data (e.g. mzML) requiring peak picking and grouping first —
  tidyMass expects a processed feature table for the data-cleaning stage
- The feature table has not yet been filtered for noisy features (features with >20%
  MVs in QC or >50% MVs in all study groups should be removed before imputation)
- The table is already complete/imputed or is destined for a non-tidyMass pipeline
  whose data structure differs from the mass_dataset class
edam_operation: http://edamontology.org/operation_3754
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: masscleaner
  role: Provides the KNN imputation method applied to the MV-filtered feature table,
    alongside the noisy-feature filter and SVR normalization
- name: massdataset
  role: Loads the processed feature table into the mass_dataset object that masscleaner
    operates on
- name: tidyMass
  role: R-based computational framework (ecosystem of R packages) within which the
    data-cleaning workflow is executed
  repo: https://github.com/tidymass/tidymass
- name: R
  role: Language in which the tidyMass data-cleaning and imputation workflow is implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:6cde49d3390b1fad1cfc3390d4a7518d79d97f09502089d33e082573ee43059a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - masscleaner
  - massdataset
  - tidyMass
  edam_operation: http://edamontology.org/operation_3754
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# knn-missing-value-imputation

## Summary

KNN-based imputation of remaining missing values (MVs) in an LC-MS untargeted metabolomics feature table, performed with the massCleaner/massdataset tidyMass workflow after noisy-feature removal and before SVR normalization.

## When to use

Use when an extracted LC-MS feature table still contains missing values after noisy features (>20% MVs in QC samples or >50% MVs across study groups) have been removed, and a complete intensity matrix is needed for statistical analysis.

## When NOT to use

- Input is raw LC-MS data (e.g. mzML) requiring peak picking and grouping first — tidyMass expects a processed feature table for the data-cleaning stage
- The feature table has not yet been filtered for noisy features (features with >20% MVs in QC or >50% MVs in all study groups should be removed before imputation)
- The table is already complete/imputed or is destined for a non-tidyMass pipeline whose data structure differs from the mass_dataset class

## Inputs

- Processed LC-MS feature/intensity table (peak-picked, grouped) loaded as a mass_dataset object via the massdataset R package
- Sample class metadata distinguishing QC samples from study groups (used by the upstream >20% MV-in-QC / >50% MV-in-groups filter)

## Outputs

- Feature table with no remaining missing values (KNN-imputed intensities), ready for SVR normalization
- Log of the number of features removed by each MV threshold prior to imputation

## How to apply

Load the case-study processed feature table into a mass_dataset object using the massdataset package in R. First apply the massCleaner noisy-feature filter, removing any feature with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups — this ordering matters because imputation is only meaningful on features that survive the MV-threshold filter. Then invoke the KNN imputation method provided by massCleaner on the filtered intensity table, so that missing intensities are estimated from the k nearest features in the sample-space rather than being dropped or zero-filled. The imputed table should subsequently feed SVR-based normalization and the statistical analysis step of the tidyMass CRC (colorectal cancer tissue) case study. Record how many features were removed at each MV threshold so the imputation input is traceable, consistent with tidyMass's uniform data format for tracing prior processing steps and parameters.

## Related tools

- **masscleaner** (Provides the KNN imputation method applied to the MV-filtered feature table, alongside the noisy-feature filter and SVR normalization)
- **massdataset** (Loads the processed feature table into the mass_dataset object that masscleaner operates on)
- **tidyMass** (R-based computational framework (ecosystem of R packages) within which the data-cleaning workflow is executed) — https://github.com/tidymass/tidymass
- **R** (Language in which the tidyMass data-cleaning and imputation workflow is implemented)

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- No remaining missing values (0 MVs) in the imputed feature table, verified with the mass_dataset missing-value summary
- The imputed table's feature count equals the count after noisy-feature removal (features removed by the >20% MV-in-QC and >50% MV-in-groups thresholds are recorded, not imputed)
- Imputed intensity values fall within plausible ranges relative to detected intensities for each feature (no zero-filling or negative/absurd intensities introduced)
- The imputed table reproduces the downstream SVR normalization summary counts when carried through the documented tidyMass case-study pipeline (docker image: jaspershen/tidymass-case-study)
- Sample metadata (QC vs study groups) is preserved unchanged after imputation

## Limitations

- KNN imputation is applied only after noisy-feature removal; applying it to features violating the MV thresholds would impute mostly-missing features instead of removing them
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, so the mass_dataset output may need conversion for external pipelines
- Thresholds (20% MVs in QC samples, 50% MVs in all study groups) are specific to the CRC case study and may not transfer to other study designs

## Evidence

- [intro] Features with more than 20% missing values in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [intro] Requiring optimization of multiple steps such as data cleaning: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment"
- [abstract] Uniform data format stores processed data and processing parameters for traceability: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [readme] Install tidymass from GitLab or GitHub: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
