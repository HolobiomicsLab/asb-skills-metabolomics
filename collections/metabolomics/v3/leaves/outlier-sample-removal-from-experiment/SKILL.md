---
name: outlier-sample-removal-from-experiment
description: Use when a lipidomics experiment (e.g. Metabolomics Workbench study imported
  into R) must be quality-controlled before differential or multivariate analysis,
  and suspect samples must be identified and dropped.
when_to_use_negative:
- Input is a raw numerical matrix or Skyline CSV that has not yet been converted to
  a LipidomicsExperiment via as_lipidomics_experiment()/read_skyline() — import and
  annotation must precede QC
- The goal is filtering measured lipids rather than samples — lipidr filters lipids
  by %CV, which is a separate operation
- Only a handful of samples exist where a PCA score plot cannot meaningfully reveal
  dispersion outliers
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: R/Bioconductor package used to run PCA via mva(measure="Area", method="PCA"),
    render interactive score plots colored by SampleType, and subset out outlier samples
    from the LipidomicsExperiment
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which the lipidr QC workflow executes; lipidr extends
    SummarizedExperiment so filtered objects stay Bioconductor-compatible
- name: Metabolomics Workbench API
  role: Source of the public lipidomics dataset (study ST001111) whose samples are
    screened for outliers
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:84da0c628514ca78d6d03e3501f0b2a584fa64cae4431dfdfa309f79e93cf812
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# outlier-sample-removal-from-experiment

## Summary

Identifies samples with abnormally large dispersion in a lipidomics dataset by inspecting a PCA score plot generated with lipidr's mva() function, then removes the flagged outliers from the LipidomicsExperiment before downstream analysis. This guards univariate and multivariate results against contamination by anomalous samples.

## When to use

Use when a lipidomics experiment (e.g. Metabolomics Workbench study imported into R) must be quality-controlled before differential or multivariate analysis, and suspect samples must be identified and dropped.

## When NOT to use

- Input is a raw numerical matrix or Skyline CSV that has not yet been converted to a LipidomicsExperiment via as_lipidomics_experiment()/read_skyline() — import and annotation must precede QC
- The goal is filtering measured lipids rather than samples — lipidr filters lipids by %CV, which is a separate operation
- Only a handful of samples exist where a PCA score plot cannot meaningfully reveal dispersion outliers

## Inputs

- LipidomicsExperiment object imported into R (e.g. from a Metabolomics Workbench study such as ST001111, name-corrected, with measure="Area" peak areas)
- Sample annotation table with variables such as SampleType for coloring PCA score plots

## Outputs

- PCA scores plot (figure/*.png) colored by SampleType highlighting outlier samples
- Cleaned LipidomicsExperiment with outlier samples removed (e.g. cleaned_st001111.rds)

## How to apply

Load the imported, name-corrected LipidomicsExperiment (e.g. from Metabolomics Workbench study ST001111) into R and run lipidr::mva() with measure="Area" and method="PCA" to compute principal components of the lipid peak areas. Plot the PCA score plot colored by a sample annotation variable such as SampleType using lipidr's interactive plotting. Inspect the scores for samples with significantly large dispersion from the main cluster — in ST001111 breast-cancer tissues this flagged samples 42 and 18. Remove those samples from the experiment, save the PCA figure and the cleaned dataset, and re-run mva() on the filtered object to confirm the remaining samples cluster as expected. Rationale: lipidr generates PCA score plots explicitly 'for quality control of samples and measured lipids', so dispersion-based outlier detection is a supported QC step prior to normalization and univariate analysis.

## Related tools

- **lipidr** (R/Bioconductor package used to run PCA via mva(measure="Area", method="PCA"), render interactive score plots colored by SampleType, and subset out outlier samples from the LipidomicsExperiment) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which the lipidr QC workflow executes; lipidr extends SummarizedExperiment so filtered objects stay Bioconductor-compatible)
- **Metabolomics Workbench API** (Source of the public lipidomics dataset (study ST001111) whose samples are screened for outliers)

## Examples

```
d <- lipidr::mva(cleaned_st001111, measure="Area", method="PCA"); plot_scores(d, "PCA", color_by="SampleType")
```

## Evaluation signals

- The PCA score plot flags a small number of samples with significantly large dispersion from the main cluster (in ST001111, exactly samples 42 and 18)
- After removal, re-running mva(method="PCA") shows remaining samples clustering by SampleType without extreme dispersion
- The cleaned LipidomicsExperiment has reduced sample (col) dimension matching the number of removed outliers
- A PCA figure is saved under figure/*.png and the cleaned dataset saved as RDS (e.g. cleaned_st001111.rds)
- Lipid feature (row) dimension is unchanged — only samples, not measured lipids, were removed

## Limitations

- Outlier identification is visual/interactive — dispersion is judged by inspecting the score plot rather than by an automated statistical cutoff
- PCA here is a quality-control view; lipidr's QC also includes %CV-based lipid filtering and normalization with/without internal standards, which are complementary and not performed by this skill
- Removal decisions depend on correct sample names and annotations — sample names must be identical in the data matrix and the annotation table

## Evidence

- [results] In the PCA scores plot of ST001111 colored by SampleType, two outlier samples with significantly large dispersion (samples `42` and `18`) were identified and removed from the dataset.: [evidence span withheld — anchor status: not_found]
- [readme] `lipidr` implements PCA, PCoA and OPLS(DA) to reveal patterns in data and discover variables related to an outcome of interest.: [evidence span withheld — anchor status: external_doc]
