---
name: summarized-experiment-object-manipulation
description: Use when a metabonomic dataset must be filtered (e.g. CV_th = 0.30 reproducibility
  filter) while keeping the feature matrix and associated metadata coherently stored
  in a single SummarizedExperiment container that downstream MWASTools functions expect.
when_to_use_negative:
- Input is already a plain feature matrix or table rather than a SummarizedExperiment
  container, so CV_filter cannot operate on the metadata-carrying object.
- A QC filter other than coefficient-of-variation reproducibility is required (e.g.
  blank subtraction or signal-to-noise filtering).
- The goal is association modeling or STOCSY assignment rather than reproducibility
  filtering of the metabo_SE container.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3172
tools:
- name: MWASTools
  role: Provides CV_filter() and the SummarizedExperiment-based pipeline the filtered
    metabo_SE object feeds into.
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which the SummarizedExperiment objects are constructed,
    filtered, and saved as RDS.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:30cbbe5941cda31794d29a2063c9e0f5a9944239f5817a009fb00a95586d1fe1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# summarized-experiment-object-manipulation

## Summary

This skill covers constructing and re-assigning MWASTools' metabo_SE SummarizedExperiment objects so that metabolomic feature matrices, phenotype metadata, and per-feature QC tables (e.g. metabo_CV) stay synchronized through the CV-filtering and downstream MWAS steps.

## When to use

Use when a metabonomic dataset must be filtered (e.g. CV_th = 0.30 reproducibility filter) while keeping the feature matrix and associated metadata coherently stored in a single SummarizedExperiment container that downstream MWASTools functions expect.

## When NOT to use

- Input is already a plain feature matrix or table rather than a SummarizedExperiment container, so CV_filter cannot operate on the metadata-carrying object.
- A QC filter other than coefficient-of-variation reproducibility is required (e.g. blank subtraction or signal-to-noise filtering).
- The goal is association modeling or STOCSY assignment rather than reproducibility filtering of the metabo_SE container.

## Inputs

- metabo_SE SummarizedExperiment object containing the metabolomic feature matrix
- metabo_CV per-feature coefficient-of-variation table produced by the upstream CV-analysis task
- CV cut-off value (e.g. CV_th = 0.30)

## Outputs

- Filtered metabo_SE SummarizedExperiment retaining only features with CV <= 0.30
- RDS file storing the filtered SummarizedExperiment object
- Record of retained feature count and their ppm values

## How to apply

Load the SummarizedExperiment object (metabo_SE) holding the metabolic feature matrix plus the per-feature CV table (metabo_CV) produced by the upstream CV-analysis task. Apply MWASTools::CV_filter(metabo_SE, metabo_CV, CV_th = 0.30) so that only features with coefficient of variation <= 0.30 are retained, and reassign the returned SummarizedExperiment to metabo_SE so the filtered container is used by subsequent MWAS steps. Finally save the filtered SummarizedExperiment as an RDS file and record the number of retained features and their ppm values.

## Related tools

- **MWASTools** (Provides CV_filter() and the SummarizedExperiment-based pipeline the filtered metabo_SE object feeds into.) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which the SummarizedExperiment objects are constructed, filtered, and saved as RDS.)

## Examples

```
metabo_SE <- CV_filter(metabo_SE, metabo_CV, CV_th = 0.30); saveRDS(metabo_SE, 'metabo_SE_filtered.rds')
```

## Evaluation signals

- Every retained feature's CV value in metabo_CV is <= 0.30.
- Every removed feature's CV value exceeds 0.30.
- Number of retained features and their ppm values are recorded after filtering.
- Filtered SummarizedExperiment is saved to disk as an RDS file and remains a valid SummarizedExperiment object for downstream MWAS functions.
- Reassignment of the filtered container to metabo_SE is reflected in subsequent steps rather than discarded.

## Limitations

- The CV cut-off (CV_th) is fixed by the user and directly determines which features survive; an inappropriately chosen threshold silently discards biologically relevant features.
- Filtering is only meaningful if metabo_CV was computed on the same features as metabo_SE; mismatched upstream tables yield inconsistent retention decisions.

## Evidence

- [other] CV_filter mechanism and CV_th cut-off: [evidence span withheld — anchor status: not_found]
- [abstract] MWASTools scope statement: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
