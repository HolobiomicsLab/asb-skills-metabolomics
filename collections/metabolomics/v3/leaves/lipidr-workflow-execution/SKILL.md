---
name: lipidr-workflow-execution
description: Use when a Metabolomics Workbench lipidomics study (e.g., ST001111, measure="Area")
  has been imported as a LipidomicsExperiment and you need to run multivariate quality
  control such as PCA to detect outlier samples.
when_to_use_negative:
- The dataset has not yet been imported and name-corrected into a LipidomicsExperiment
  — run the import/name-correction step first
- The goal is differential or enrichment analysis (limma-based univariate analysis
  or LSEA) rather than sample-level quality control
- Input is raw Skyline instrument output not yet exported as CSV or converted to a
  numerical matrix with a matching sample-annotation table
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: 'Core R/Bioconductor package: imports the Metabolomics Workbench study, runs
    mva(measure="Area", method="PCA"), and renders interactive score plots for outlier
    identification'
  repo: https://github.com/ahmohamed/lipidr
- name: Metabolomics Workbench API
  role: Source of the public lipidomics dataset (ST001111) that lipidr searches, downloads,
    and reanalyzes
- name: R/Bioconductor
  role: Runtime environment in which the lipidr workflow and LipidomicsExperiment
    (extending SummarizedExperiment) objects are executed
  repo: https://github.com/ahmohamed/lipidr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:a64a38353d028488f7c70ce16abe9a819e76b2af5ab0606881ce0f3385c9d78f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ahmohamed/lipidr
  tools:
  - Metabolomics Workbench API
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lipidr-workflow-execution

## Summary

Runs the lipidr R/Bioconductor workflow on Metabolomics Workbench lipidomics studies (e.g., ST001111 breast-cancer tissues), from import and name correction through multivariate analysis (PCA) with interactive plotting, to identify and remove outlier samples before downstream analysis.

## When to use

Use when a Metabolomics Workbench lipidomics study (e.g., ST001111, measure="Area") has been imported as a LipidomicsExperiment and you need to run multivariate quality control such as PCA to detect outlier samples.

## When NOT to use

- The dataset has not yet been imported and name-corrected into a LipidomicsExperiment — run the import/name-correction step first
- The goal is differential or enrichment analysis (limma-based univariate analysis or LSEA) rather than sample-level quality control
- Input is raw Skyline instrument output not yet exported as CSV or converted to a numerical matrix with a matching sample-annotation table

## Inputs

- Imported, name-corrected LipidomicsExperiment object from Metabolomics Workbench study ST001111 (lipid peak areas, measure="Area")
- Sample annotation / group table (e.g., SampleType) with sample names identical to the experiment
- Skyline CSV export or numerical matrix (lipids as rows, samples as columns) as alternative lipidr input formats

## Outputs

- PCA scores figure saved as figure/*.png
- Cleaned experiment object saved as cleaned_st001111.rds (outlier samples 42 and 18 removed)
- Interactive PCA score plot colored by SampleType

## How to apply

Load the imported, name-corrected ST001111 LipidomicsExperiment into R (lipidr imports Metabolomics Workbench files directly, automatically inferring lipid class and chain information). Run lipidr::mva(measure="Area", method="PCA") to compute a principal component analysis of the lipid peak areas, then use lipidr's interactive plotting to draw the PCA score plot colored by SampleType. Inspect the score plot for samples with significantly large dispersion — for ST001111 this flags samples 42 and 18, which should be removed before further analysis. Save the PCA scores figure (figure/*.png) and the cleaned dataset (cleaned_st001111.rds). This mirrors lipidr's overall design of thorough data inspection, normalization, and uni- and multivariate analyses displayed as interactive visualizations.

## Related tools

- **lipidr** (Core R/Bioconductor package: imports the Metabolomics Workbench study, runs mva(measure="Area", method="PCA"), and renders interactive score plots for outlier identification) — https://github.com/ahmohamed/lipidr
- **Metabolomics Workbench API** (Source of the public lipidomics dataset (ST001111) that lipidr searches, downloads, and reanalyzes)
- **R/Bioconductor** (Runtime environment in which the lipidr workflow and LipidomicsExperiment (extending SummarizedExperiment) objects are executed) — https://github.com/ahmohamed/lipidr

## Examples

```
library(lipidr); d <- read_skyline("Skyline_export.csv"); d <- add_sample_annotation(d, "clin.csv"); mva(d, measure = "Area", method = "PCA")
```

## Evaluation signals

- The mva call uses measure="Area" and method="PCA" on the ST001111 LipidomicsExperiment without error
- The PCA score plot is colored by SampleType and interactively explorable in lipidr's plotting
- Samples 42 and 18 appear as the two points with significantly large dispersion and are removed from the experiment
- The cleaned dataset is written to cleaned_st001111.rds and the scores figure to figure/*.png
- The saved RDS no longer contains samples 42 and 18 in its sample annotations

## Limitations

- Outlier identification is visual/interactive — the two high-dispersion samples are judged by inspecting the score plot rather than by a numeric cutoff
- Requires sample names to be identical between the data matrix and the sample-annotation file, otherwise add_sample_annotation fails
- lipidr is targeted at lipidomics peak-area data with internal-standard-aware normalization; general metabolomics peak tables outside its import formats are not directly supported

## Evidence

- [readme] `lipidr` implements PCA, PCoA and OPLS(DA) to reveal patterns in data and discover variables related to an outcome of interest.: [evidence span withheld — anchor status: external_doc]
