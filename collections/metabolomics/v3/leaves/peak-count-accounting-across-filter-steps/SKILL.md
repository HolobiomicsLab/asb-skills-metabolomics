---
name: peak-count-accounting-across-filter-steps
description: 'Decision support for choosing a Sample Filtering percentage in Galaxy-M:
  re-run the DIMS (100% threshold) and LC-MS (80% threshold) pipelines over MTBLS79
  and MTBLS146 data, and record per-step peak removals before deciding whether Missing
  Values Imputation (e.g. KNN) is required.'
when_to_use_negative:
- The pipeline's Sample Filtering threshold is already fixed and the downstream imputation
  decision has been made — there is nothing to re-run or compare.
- Peak counts are needed from XCMS feature detection or SIM-stitching itself rather
  than from the Replicate/Blank/Sample Filtering stages.
- Data contains no blank or replicate metadata (class labels for biological vs blank
  samples), so the DSO-based filter chain cannot be evaluated meaningfully.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M
  role: Hosts the DIMS and LC-MS workflows whose Replicate/Blank/Sample Filtering
    steps are re-run and whose peak counts are tabulated
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform in which the published workflow histories are loaded and the filtering
    steps are executed to recover peak counts
  repo: https://github.com/galaxyproject/galaxy
- name: XCMS
  role: Produces the aligned LC-MS feature table that enters the filtering chain for
    the LC-MS case study
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:6afad18bcdf00dec4aadaf0c41e83bb4e53cc788e662e3af8cc7309ded7731dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peak-count-accounting-across-filter-steps

## Summary

Tracks how many peaks enter and leave each Galaxy-M filtering stage (Replicate Filtering, Blank Filtering, Sample Filtering) for the DIMS and LC-MS case studies, so that the imputation requirement at the end of the pipeline can be attributed to a specific Sample Filtering threshold (100% vs 80%).

## When to use

Decision support for choosing a Sample Filtering percentage in Galaxy-M: re-run the DIMS (100% threshold) and LC-MS (80% threshold) pipelines over MTBLS79 and MTBLS146 data, and record per-step peak removals before deciding whether Missing Values Imputation (e.g. KNN) is required.

## When NOT to use

- The pipeline's Sample Filtering threshold is already fixed and the downstream imputation decision has been made — there is nothing to re-run or compare.
- Peak counts are needed from XCMS feature detection or SIM-stitching itself rather than from the Replicate/Blank/Sample Filtering stages.
- Data contains no blank or replicate metadata (class labels for biological vs blank samples), so the DSO-based filter chain cannot be evaluated meaningfully.

## Inputs

- Published Galaxy-M workflow histories for the DIMS and LC-MS pipelines from the pre-installed VM
- DIMS FT-ICR MS test data (SIM-stitched .dat-derived peak lists) from ~/GalaxyM-TestData/DIMS_DATA (MetaboLights MTBLS79)
- LC-MS test data processed by XCMS-Basic-Birmingham-Pipe (feature detection, RT correction, alignment) from ~/GalaxyM-TestData/LCMS_DATA (MetaboLights MTBLS146)
- User-specified Sample Filtering x-out-of-n threshold (100% for DIMS, 80% for LC-MS)

## Outputs

- Per-filter-stage peak counts (peaks_in, peaks_removed, peaks_out) for Replicate Filtering, Blank Filtering, and Sample Filtering in each case study
- Record of whether Missing Values Imputation was executed per case study (none for DIMS at 100%; KNN for LC-MS at 80%)
- filter_imputation_comparison.csv with one row per case study listing thresholds, removed-peak counts per filter, and imputation usage

## How to apply

Load the two pre-installed Galaxy-M workflow histories from the GigaDB VM (test data in ~/GalaxyM-TestData/DIMS_DATA from MetaboLights MTBLS79 and ~/GalaxyM-TestData/LCMS_DATA from MTBLS146). Re-run each pipeline's filtering chain — Replicate Filtering (x-out-of-n technical replicates), Blank Filtering (peaks as strong in blanks as biological samples), and Sample Filtering (x-out-of-n biological samples) — using the DIMS 100% threshold and the LC-MS 80% threshold as published. At each stage record peaks_in, peaks_removed, and peaks_out, and verify peaks_out = peaks_in − peaks_removed for every filter step. Then check whether Missing Values Imputation was executed: under the 100% threshold no imputation is needed, whereas the 80% LC-MS threshold leaves missing values that require KNN imputation before statistical preparation. Assemble the thresholds, removed-peak counts, and imputation usage into one row per case study (e.g. filter_imputation_comparison.csv).

## Related tools

- **Galaxy-M** (Hosts the DIMS and LC-MS workflows whose Replicate/Blank/Sample Filtering steps are re-run and whose peak counts are tabulated) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform in which the published workflow histories are loaded and the filtering steps are executed to recover peak counts) — https://github.com/galaxyproject/galaxy
- **XCMS** (Produces the aligned LC-MS feature table that enters the filtering chain for the LC-MS case study)

## Examples

```
galaxym run_workflow --workflow LCMS_Birmingham_Pipe --data ~/GalaxyM-TestData/LCMS_DATA --sample_filter 80
```

## Evaluation signals

- peaks_out = peaks_in − peaks_removed holds for every filter step in both case studies
- Recorded Sample Filtering thresholds match the stored published Galaxy-M workflow histories (100% for DIMS, 80% for LC-MS)
- Imputation usage column agrees with the histories: no imputation for DIMS at 100%, Missing Values Imputation executed for LC-MS at 80%
- filter_imputation_comparison.csv has exactly one row per case study (DIMS and LC-MS) with non-negative removed-peak counts
- Removed-peak counts decrease monotonically across Replicate Filtering → Blank Filtering → Sample Filtering within each pipeline

## Limitations

- The published DIMS transient (.dat) files were not in the MetaboLights accession and had to be obtained from the original authors; without them the DIMS filtering chain cannot be re-run.
- The LC-MS workflow implemented (XCMS-Basic-Birmingham-Pipe) had not been used previously in published work and is only a representative series of processing steps, so the peak-removal numbers may not generalize to other LC-MS pipelines.
- Results are specific to the two case studies (MTBLS79 DIMS, MTBLS146 LC-MS); no general guidance on threshold choice beyond the two published values is provided.

## Evidence

- [methods] Sample Filtering threshold determines imputation requirement: [evidence span withheld — anchor status: not_found]
- [abstract] Sample Filtering removes peaks failing x-out-of-n samples: [evidence span withheld — anchor status: not_found]
- [methods] Test data are subsets of MetaboLights accessions: "The test data sets (both LCMS and DIMS) can be obtained from their larger data sets in Metabolights (DIMS: Accession MTBLS79)"
- [readme] VM holds test data in GalaxyM-TestData directories: "the Mass Spectra can be found in : ~/GalaxyM-TestData/LCMS_DATA ~/GalaxyM-TestData/DIMS_DATA"
- [abstract] LC-MS pipeline groups/aligns features with XCMS: "groups/aligns spectral features across the samples using the R-package XCMS"
