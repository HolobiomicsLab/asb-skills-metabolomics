---
name: sample-annotation-integration
description: Use when a lipidomics dataset (Skyline CSV export or numerical matrix)
  needs its clinical/group metadata merged in before normalization, de_analysis, or
  plotting — e.g. linking diet groups to Skyline-exported samples.
when_to_use_negative:
- Sample names in the annotation table do not exactly match the sample names in the
  measurement file — fix the naming first rather than merging, since lipidr requires
  identical names
- The input is an untargeted feature table (mz/rt features) without lipid names —
  lipidr's automatic inference of lipid class and chain information from lipid names
  will not apply
- No group/clinical variable is needed downstream (e.g. purely QC-oriented inspection
  where no differential analysis or grouped visualization is planned)
edam_topics:
- http://edamontology.org/topic_3311
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: Provides read_skyline/as_lipidomics_experiment and add_sample_annotation to
    merge clinical metadata into the LipidomicsExperiment
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment; the annotated object extends Bioconductor's SummarizedExperiment
    for integration with other packages
- name: Skyline
  role: Source of the exported results CSV whose sample names must match the clinical
    annotation table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:4ad738f052bdad5f77544bb119c273f6e992d93ce54322ebdfc33fe8549f7ac6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R/Bioconductor
  - Skyline
  - lipidr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3311
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sample-annotation-integration

## Summary

Attaches a sample annotation/clinical table (e.g. Skyline clin.csv) to a lipidr LipidomicsExperiment so that group variables such as diet condition are available for downstream differential analysis and volcano plots.

## When to use

Use when a lipidomics dataset (Skyline CSV export or numerical matrix) needs its clinical/group metadata merged in before normalization, de_analysis, or plotting — e.g. linking diet groups to Skyline-exported samples.

## When NOT to use

- Sample names in the annotation table do not exactly match the sample names in the measurement file — fix the naming first rather than merging, since lipidr requires identical names
- The input is an untargeted feature table (mz/rt features) without lipid names — lipidr's automatic inference of lipid class and chain information from lipid names will not apply
- No group/clinical variable is needed downstream (e.g. purely QC-oriented inspection where no differential analysis or grouped visualization is planned)

## Inputs

- Skyline-exported results CSV (e.g. inst/extdata/A1_data.csv) or numerical matrix CSV with lipids as rows and samples as columns
- Sample annotation / clinical table CSV (e.g. clin.csv) with sample names in the first column

## Outputs

- Annotated LipidomicsExperiment (extends SummarizedExperiment) with clinical variables (e.g. diet group) available in colData for downstream de_analysis and volcano plotting

## How to apply

Import the measurement data first — read_skyline() for a Skyline-exported CSV (transition-level results) or as_lipidomics_experiment() for a numerical matrix with lipids as rows and samples as columns — then attach the annotation file with add_sample_annotation(), passing the clin.csv path (Skyline workflow) or the clinical CSV (numerical-matrix workflow). The annotation table must have sample names in its first column, and every sample name must be identical to those in the measurement data; lipidr stores the result as a LipidomicsExperiment extending SummarizedExperiment so clinical variables land in the colData. After annotation, summarize transitions to the lipid-molecule level and normalize (e.g. PQN) before running de_analysis on any loaded clinical variable — in the vignette workflow the attached diet variable drives the HighFat_water vs NormalDiet_water contrast with measure='Area'. Verify success by confirming the clinical variables are retrievable from the annotated object and usable to construct the differential-analysis design.

## Related tools

- **lipidr** (Provides read_skyline/as_lipidomics_experiment and add_sample_annotation to merge clinical metadata into the LipidomicsExperiment) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment; the annotated object extends Bioconductor's SummarizedExperiment for integration with other packages)
- **Skyline** (Source of the exported results CSV whose sample names must match the clinical annotation table)

## Examples

```
d <- read_skyline("Skyline_export.csv"); d <- add_sample_annotation(d, "data_clin.csv")
```

## Evaluation signals

- Every sample name in the annotation CSV is identical (string-for-string) to a sample name in the Skyline export or matrix — lipidr requires this
- The returned object is a LipidomicsExperiment extending SummarizedExperiment and the clinical variables (e.g. diet) are retrievable from its colData
- Sample order/dimension of the annotated object matches the input measurement data (annotation adds metadata, not rows/columns)
- Downstream de_design/de_analysis can reference the attached variable — e.g. the vignette contrast HighFat_water - NormalDiet_water with measure='Area' runs without missing-factor errors
- Volcano plots and grouped plots can be generated using the loaded clinical variables

## Limitations

- Sample names in the two files must be identical; mismatched names are an explicit failure condition stated in the README
- The annotation step alone does not normalize or summarize data — transition-level Skyline measurements still need summarize_transitions and PQN normalization before de_analysis
- Only samples present in both files can be annotated; extra samples in either file will cause integration problems

## Evidence

- [readme] A table / CSV file with the sample annotation / groups, where the sample names are in first column. Note the sample names must be identical in the two files.: [evidence span withheld — anchor status: external_doc]
- [readme] LipidomicsExperiment extends SummarizedExperiment: [evidence span withheld — anchor status: external_doc]
