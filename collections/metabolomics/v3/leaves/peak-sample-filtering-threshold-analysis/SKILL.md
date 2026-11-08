---
name: peak-sample-filtering-threshold-analysis
description: Decision support for choosing a Sample Filtering threshold in a Galaxy-M
  DIMS or LC-MS workflow, when you need to anticipate whether Missing Values Imputation
  (e.g. KNN) will be required afterwards.
when_to_use_negative:
- The input is already an imputed or fully filtered feature table with no per-step
  filtering metadata to compare
- You only need to run a single fixed Sample Filtering threshold with no interest
  in the threshold-imputation trade-off
- The workflow contains no Sample Filtering or Missing Values Imputation steps (e.g.
  a bare XCMS feature-detection-only run)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Galaxy-M
  role: Platform hosting the published DIMS and LC-MS workflows whose Sample Filtering
    thresholds and imputation steps are compared
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Workflow engine in which the histories are re-run and the Missing Values Imputation
    step is invoked or skipped
  repo: https://github.com/galaxyproject/galaxy
- name: XCMS
  role: R-package used in the LC-MS pipeline (XCMS-Basic-Birmingham-Pipe) whose feature
    table feeds the Sample Filtering step under test
  repo: https://metlin.scripps.edu/xcms/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:1ff1afdfde58d1adf57a3f982de413998d4ca076a67e2baea0066be4e3b15f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# peak-sample-filtering-threshold-analysis

## Summary

A diagnostic skill for determining how the Sample Filtering threshold (x-out-of-n biological samples) drives the need for Missing Values Imputation in Galaxy-M metabolomics workflows. It links filter cutoff choice to downstream data-cleansing requirements in DIMS and LC-MS pipelines.

## When to use

Decision support for choosing a Sample Filtering threshold in a Galaxy-M DIMS or LC-MS workflow, when you need to anticipate whether Missing Values Imputation (e.g. KNN) will be required afterwards.

## When NOT to use

- The input is already an imputed or fully filtered feature table with no per-step filtering metadata to compare
- You only need to run a single fixed Sample Filtering threshold with no interest in the threshold-imputation trade-off
- The workflow contains no Sample Filtering or Missing Values Imputation steps (e.g. a bare XCMS feature-detection-only run)

## Inputs

- Published Galaxy-M workflow histories (DIMS and LC-MS case studies, pre-installed in the Galaxy-M VM)
- DIMS test data from ~/GalaxyM-TestData/DIMS_DATA (MetaboLights MTBLS79 subset, with transient .dat files from GigaDB)
- LC-MS test data from ~/GalaxyM-TestData/LCMS_DATA (MetaboLights MTBLS146 subset)
- User-chosen Sample Filtering thresholds (e.g. 100% and 80% of biological samples)

## Outputs

- filter_imputation_comparison.csv with one row per case study: filter thresholds, peaks removed at each filtering step, and imputation usage
- Per-filter step counts (peaks_in, peaks_removed, peaks_out) for Replicate Filtering, Blank Filtering, and Sample Filtering
- Assessment of whether Missing Values Imputation (e.g. KNN) was executed per case study

## How to apply

In Galaxy-M, re-run the published DIMS and LC-MS histories with Sample Filtering set to two contrasting thresholds: 100% (a peak must appear in all biological samples) versus 80%. Record peaks entering and leaving Replicate Filtering, Blank Filtering, and Sample Filtering, then check for each case whether Missing Values Imputation was executed — the DIMS 100% case needed no imputation, while the LC-MS 80% case required KNN imputation because missing values remained. Validate that peaks_out = peaks_in - peaks_removed for every filter step in each case study. Tabulate thresholds, removed-peak counts per filter, and imputation usage per case study to expose the threshold-imputation relationship.

## Related tools

- **Galaxy-M** (Platform hosting the published DIMS and LC-MS workflows whose Sample Filtering thresholds and imputation steps are compared) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Workflow engine in which the histories are re-run and the Missing Values Imputation step is invoked or skipped) — https://github.com/galaxyproject/galaxy
- **XCMS** (R-package used in the LC-MS pipeline (XCMS-Basic-Birmingham-Pipe) whose feature table feeds the Sample Filtering step under test) — https://metlin.scripps.edu/xcms/

## Examples

```
re-run Galaxy-M histories: DIMS with Sample Filtering x=n (100%) and LC-MS with x=0.8n (80%); record peaks_in/removed/out per filter; check Missing Values Imputation usage; write filter_imputation_comparison.csv
```

## Evaluation signals

- Internal consistency: for every filter step in each case study, peaks_out = peaks_in - peaks_removed
- Tabulated Sample Filtering thresholds (100% DIMS, 80% LC-MS) match the stored published workflow histories
- Imputation usage matches the histories: no imputation in the DIMS 100% case; KNN Missing Values Imputation in the LC-MS 80% case
- Both case studies complete on the pre-installed Galaxy-M VM using data from ~/GalaxyM-TestData/DIMS_DATA and ~/GalaxyM-TestData/LCMS_DATA
- The resulting CSV contains one row per case study with thresholds, per-filter removed-peak counts, and imputation fields populated

## Limitations

- Sample Filtering thresholds are user-chosen, so results are case-study-specific and may not generalize to other metabolomics datasets
- The missing-value–imputation link observed in these two case studies is inferential; the article does not provide a parameter sweep of intermediate thresholds
- No univariate statistics tools are included in the current implementation, so downstream statistical impact of the threshold choice cannot be assessed within Galaxy-M
- PCA output is non-graphical, so visual checks of filtering effects are not supported
- Missing-values-free data after 100% filtering depends on the complete SIM-stitched DIMS peak matrix; sparse LC-MS alignment outputs will still yield missing values at high thresholds

## Evidence

- [abstract] Sample Filtering threshold drives imputation need: [evidence span withheld — anchor status: not_found]
- [abstract] Data cleansing includes missing value imputation: "The range of tools presented spans from processing of raw data, e.g. peak picking and alignment, through data cleansing, e.g. missing value imputation, to preparation for statistical analysis"
- [readme] Sample Filtering parameters stored in workflows: "The pipelines/workflows also make use of CSV files for certain instructions and these can be found in the GalaxyM-TestData folder mentioned above. See the published histories or workflows for where"
- [readme] Test data location on VM: "the Mass Spectra can be found in :

* `~/GalaxyM-TestData/LCMS_DATA`
* `~/GalaxyM-TestData/DIMS_DATA`"
