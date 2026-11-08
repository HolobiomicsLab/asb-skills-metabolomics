---
name: dims-lcms-case-study-reproduction
description: Decision support for choosing Sample Filtering thresholds in Galaxy-M
  metabolomics pipelines when planning whether downstream Missing Values Imputation
  (e.g. KNN) will be needed after filtering removes peaks with missing observations.
when_to_use_negative:
- Input is already an imputed, filtered feature table — the threshold-vs-imputation
  relationship cannot be observed without re-running the filtering steps
- Analysis targets a single pipeline only — the skill's rationale depends on comparing
  the DIMS 100% threshold case against the LC-MS 80% threshold case
- Working outside the Galaxy-M VM without the published histories — the validation
  requires matching tabulated values against the stored workflow histories
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M
  role: Workflow platform hosting the DIMS and LC-MS pipelines, filter tools, and
    Missing Values Imputation being compared
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform in which the published case-study histories are re-run and filter
    thresholds are varied
  repo: https://github.com/galaxyproject/galaxy
- name: XCMS
  role: Executes the LC-MS pipeline (XCMS-Basic-Birmingham-Pipe) for feature detection,
    retention time correction and alignment before filtering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:4ea64e73697b8a680e0e5c3fe823ccfe83f9aa9ae83e813f83d0a8d837e7a8ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - XCMS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dims-lcms-case-study-reproduction

## Summary

Reproduces the Galaxy-M DIMS and LC-MS case-study workflows in the pre-installed Galaxy-M VM to quantify how Sample Filtering thresholds determine whether Missing Values Imputation is required.

## When to use

Decision support for choosing Sample Filtering thresholds in Galaxy-M metabolomics pipelines when planning whether downstream Missing Values Imputation (e.g. KNN) will be needed after filtering removes peaks with missing observations.

## When NOT to use

- Input is already an imputed, filtered feature table — the threshold-vs-imputation relationship cannot be observed without re-running the filtering steps
- Analysis targets a single pipeline only — the skill's rationale depends on comparing the DIMS 100% threshold case against the LC-MS 80% threshold case
- Working outside the Galaxy-M VM without the published histories — the validation requires matching tabulated values against the stored workflow histories

## Inputs

- Galaxy-M published workflow histories for the DIMS and LC-MS case studies (pre-installed in the Galaxy-M VM)
- DIMS test data from ~/GalaxyM-TestData/DIMS_DATA (MetaboLights MTBLS79 subset plus transient .dat files from GigaDB, accession 10.5524/100146 or 10.5524/100108)
- LC-MS test data from ~/GalaxyM-TestData/LCMS_DATA (MetaboLights MTBLS146 subset)
- CSV instruction files in the GalaxyM-TestData folder referenced by the published workflows

## Outputs

- filter_imputation_comparison.csv with one row per case study listing filter thresholds, removed-peak counts per filter step (Replicate, Blank, Sample Filtering), and imputation usage
- Per-filter peak accounting (peaks_in, peaks_removed, peaks_out) for both pipelines
- Record of which case study executed Missing Values Imputation (DIMS: none; LC-MS: KNN imputation)

## How to apply

Load the two published Galaxy-M workflow histories from the pre-installed Galaxy-M VM (downloaded from GigaDB), using DIMS test data from ~/GalaxyM-TestData/DIMS_DATA (MetaboLights MTBLS79 subset, with transient files from GigaDB) and LC-MS data from ~/GalaxyM-TestData/LCMS_DATA (MetaboLights MTBLS146 subset). Re-run the DIMS pipeline's Replicate Filtering, Blank Filtering and Sample Filtering steps with Sample Filtering at the 100% threshold (a peak must appear in all biological samples), then re-run the LC-MS pipeline (XCMS-Basic-Birmingham-Pipe: feature detection, retention time correction and alignment) with Sample Filtering at the 80% threshold, recording peaks entering and leaving each filter. Record whether Missing Values Imputation runs: DIMS needs none under the 100% threshold because no missing values remain, while LC-MS applies KNN imputation under the 80% threshold. Assemble one row per case study into filter_imputation_comparison.csv, checking peaks_out = peaks_in - peaks_removed at every filter step against the stored histories.

## Related tools

- **Galaxy-M** (Workflow platform hosting the DIMS and LC-MS pipelines, filter tools, and Missing Values Imputation being compared) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform in which the published case-study histories are re-run and filter thresholds are varied) — https://github.com/galaxyproject/galaxy
- **XCMS** (Executes the LC-MS pipeline (XCMS-Basic-Birmingham-Pipe) for feature detection, retention time correction and alignment before filtering)

## Evaluation signals

- peaks_out = peaks_in - peaks_removed holds for every filter step (Replicate, Blank, Sample Filtering) in each case study
- Tabulated filter thresholds and imputation usage match the stored published workflow histories
- DIMS case under the 100% Sample Filtering threshold shows no Missing Values Imputation executed; LC-MS case under the 80% threshold shows KNN imputation executed
- filter_imputation_comparison.csv contains exactly one row per case study with the threshold, per-filter removed-peak counts, and imputation columns populated
- VM login (user galaxym / Galaxy user <maintainer e-mail, see the repository README>) reproduces the two stored/published workflows without reconfiguration

## Limitations

- DIMS transient (.dat) files are not included in the MetaboLights MTBLS79 accession and must be obtained separately (GigaDB) or the DIMS re-run cannot start
- The LC-MS workflow (XCMS-Basic-Birmingham-Pipe) was included as a representative series of processing steps and had not been used previously in published work
- The GigaDB VM accession was 'to follow' at time of writing, and the README warns that recreating the VM from installation instructions is complex and can involve troubleshooting (e.g. WINE must be forced to 32-bit mode to avoid 'Invalid file identifier' errors in SimStitch)

## Evidence

- [abstract] Sample Filtering: removes peaks failing to appear in x-out-of-n samples: [evidence span withheld — anchor status: not_found]
- [abstract] missing value imputation as data cleansing: "The range of tools presented spans from processing of raw data, e.g. peak picking and alignment, through data cleansing, e.g. missing value imputation, to preparation for statistical analysis"
- [abstract] LC-MS processing via XCMS-Basic-Birmingham-Pipe: "The LC-MS data is processed using XCMS [23], including feature detection, retention time correction and alignment"
- [readme] test data locations in VM: "the Mass Spectra can be found in :

* `~/GalaxyM-TestData/LCMS_DATA`
* `~/GalaxyM-TestData/DIMS_DATA`"
- [readme] two workflows stored/published in VM: "The GalaxyM installation has a user already registered and two workflows have been stored/published."
