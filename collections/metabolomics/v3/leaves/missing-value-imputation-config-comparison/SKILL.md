---
name: missing-value-imputation-config-comparison
description: Decision support for choosing a Sample Filtering threshold (x-out-of-n
  biological samples) in Galaxy-M pipelines and predicting whether Missing Values
  Imputation (e.g. KNN) will be needed downstream.
when_to_use_negative:
- The input is already a filtered/imputed peak matrix with no Sample Filtering step
  to re-run, so there is no threshold-vs-imputation relationship to test
- The analysis requires univariate statistics, which the current Galaxy-M implementation
  does not include
- The user cannot access the published workflow histories or the pre-installed VM,
  since the comparison must match the stored histories rather than ad hoc re-analyses
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Galaxy-M
  role: Platform hosting the DIMS and LC-MS pipelines (Replicate Filtering, Blank
    Filtering, Sample Filtering, Missing Values Imputation) whose published histories
    are re-run for the comparison
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: XCMS
  role: Performs LC-MS feature detection, retention time correction and alignment
    (XCMS-Basic-Birmingham-Pipe) upstream of the 80% Sample Filtering step under study
  repo: https://metlin.scripps.edu/xcms/
- name: Galaxy
  role: Platform in which the published workflows and histories are stored, re-run,
    and inspected to record filter thresholds and imputation usage
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:0b2892056576af9001384a0085cb3af946ea295d4926f6e4900b08b6e7604899
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# missing-value-imputation-config-comparison

## Summary

Compares Sample Filtering thresholds (100% vs 80% of biological samples) across the Galaxy-M DIMS and LC-MS case studies to explain why only the LC-MS workflow required the Missing Values Imputation step (KNN).

## When to use

Decision support for choosing a Sample Filtering threshold (x-out-of-n biological samples) in Galaxy-M pipelines and predicting whether Missing Values Imputation (e.g. KNN) will be needed downstream.

## When NOT to use

- The input is already a filtered/imputed peak matrix with no Sample Filtering step to re-run, so there is no threshold-vs-imputation relationship to test
- The analysis requires univariate statistics, which the current Galaxy-M implementation does not include
- The user cannot access the published workflow histories or the pre-installed VM, since the comparison must match the stored histories rather than ad hoc re-analyses

## Inputs

- Published Galaxy-M workflow histories loaded from the pre-installed Galaxy-M VM
- DIMS test data from ~/GalaxyM-TestData/DIMS_DATA (MetaboLights MTBLS79; transient files from GigaDB)
- LC-MS test data from ~/GalaxyM-TestData/LCMS_DATA (MetaboLights MTBLS146)
- CSV instruction files in the GalaxyM-TestData folder referenced by the pipelines

## Outputs

- filter_imputation_comparison.csv with one row per case study listing filter thresholds, removed-peak counts per filter step, and imputation usage

## How to apply

Load the two published Galaxy-M workflow histories from the pre-installed Galaxy-M VM (login <maintainer e-mail, see the repository README>), using DIMS test data from ~/GalaxyM-TestData/DIMS_DATA (MetaboLights MTBLS79 plus transient files from GigaDB) and LC-MS test data from ~/GalaxyM-TestData/LCMS_DATA (MetaboLights MTBLS146). Re-run the DIMS pipeline's filtering steps (Replicate Filtering, Blank Filtering, Sample Filtering) with Sample Filtering set to the 100% threshold, meaning a peak must appear in all biological samples, and record peak counts entering and leaving each filter. Re-run the LC-MS pipeline (XCMS-Basic-Birmingham-Pipe feature detection, retention time correction and alignment) with Sample Filtering set to the 80% threshold and record peaks removed at each step. For each case study, capture whether Missing Values Imputation was executed: DIMS used no imputation under the 100% threshold, while LC-MS applied KNN imputation under the 80% threshold, because the 80% rule leaves missing values in the filtered peak matrix. Assemble one row per case study into filter_imputation_comparison.csv and validate internal consistency.

## Related tools

- **Galaxy-M** (Platform hosting the DIMS and LC-MS pipelines (Replicate Filtering, Blank Filtering, Sample Filtering, Missing Values Imputation) whose published histories are re-run for the comparison) — https://github.com/Viant-Metabolomics/Galaxy-M
- **XCMS** (Performs LC-MS feature detection, retention time correction and alignment (XCMS-Basic-Birmingham-Pipe) upstream of the 80% Sample Filtering step under study) — https://metlin.scripps.edu/xcms/
- **Galaxy** (Platform in which the published workflows and histories are stored, re-run, and inspected to record filter thresholds and imputation usage) — https://github.com/galaxyproject/galaxy

## Examples

```
git clone https://github.com/galaxyproject/galaxy/; sh run.sh
```

## Evaluation signals

- Internal consistency: peaks_out = peaks_in - peaks_removed must hold for every filter step (Replicate, Blank, Sample Filtering) in each case study
- DIMS case study under the 100% Sample Filtering threshold must show no Missing Values Imputation step executed
- LC-MS case study under the 80% Sample Filtering threshold must show KNN Missing Values Imputation executed
- Tabulated thresholds and imputation usage must match the stored published workflow histories from the Galaxy-M VM
- filter_imputation_comparison.csv contains exactly one row per case study with filter thresholds, removed-peak counts, and imputation usage populated

## Limitations

- DIMS transient (.dat) files are not included in the MetaboLights accession and must be obtained from the original authors or GigaDB
- GigaDB accession details for the VM were 'to follow' at the time of writing, so VM availability may be an issue
- The Galaxy-M tool set is pinned to specific Galaxy and MI-Pack commits (Galaxy commit c429777c93680dcee449fe410f5360afbe673758; MI-Pack commit 06ce0ace643ee1cf1d27550769a5272b7ea50825), so re-runs on other versions may differ

## Evidence

- [abstract] Sample Filtering: removes peaks that fail to appear in x-out-of-n samples (x chosen by user, n = number of biological samples): [evidence span withheld — anchor status: not_found]
- [abstract] Workflow spans raw processing through data cleansing including missing value imputation: "The range of tools presented spans from processing of raw data, e.g. peak picking and alignment, through data cleansing, e.g. missing value imputation"
- [methods] Test datasets are subsets of MetaboLights accessions: "The test data sets (both LCMS and DIMS) can be obtained from their larger data sets in Metabolights (DIMS: Accession MTBLS79)"
- [readme] VM ships with registered user and two stored/published workflows: "The GalaxyM installation has a user already registered and two workflows have been stored/published."
