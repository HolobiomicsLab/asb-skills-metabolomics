---
name: r-bioconductor-lipidomics-workflow
description: Use when a lipidomics dataset (Skyline CSV export, numerical lipid-by-sample
  matrix, or Metabolomics Workbench study) needs QC, normalization, differential or
  multivariate analysis with lipid class/chain annotations in R.
when_to_use_negative:
- Input is raw vendor mass-spectrometry output rather than a Skyline export CSV, numerical
  matrix, or Metabolomics Workbench file — lipidr imports these specific formats only.
- Lipid feature names are not in parseable lipid nomenclature, since lipidr infers
  class and chain information automatically from lipid names.
- Sample names differ between the data matrix and the annotation table — the two files
  must share identical sample identifiers.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: 'Core R/Bioconductor package implementing the whole workflow: import, annotation,
    QC, normalization, univariate/multivariate analysis, and LSEA'
  repo: github.com/ahmohamed/lipidr
- name: Skyline
  role: Source of targeted lipidomics CSV exports that lipidr reads as input
- name: Metabolomics Workbench API
  role: Used by lipidr to search, download, and reanalyze public lipidomics datasets
    such as ST001111
- name: limma
  role: Referenced for users building more complex design matrices for differential
    analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:6f0720b2d779d5689ee8916680ad4f4828bda02bbe81f0e3bbebf53de4b031df
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R/Bioconductor
  - Skyline
  - limma
  - lipidr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 22
---

# r-bioconductor-lipidomics-workflow

## Summary

A complete targeted/untargeted lipidomics data-mining workflow in R/Bioconductor using the lipidr package: import and annotate lipid matrices, perform QC and normalization, then run univariate, multivariate (PCA, PCoA, OPLS-DA), and lipid set enrichment analyses.

## When to use

Use when a lipidomics dataset (Skyline CSV export, numerical lipid-by-sample matrix, or Metabolomics Workbench study) needs QC, normalization, differential or multivariate analysis with lipid class/chain annotations in R.

## When NOT to use

- Input is raw vendor mass-spectrometry output rather than a Skyline export CSV, numerical matrix, or Metabolomics Workbench file — lipidr imports these specific formats only.
- Lipid feature names are not in parseable lipid nomenclature, since lipidr infers class and chain information automatically from lipid names.
- Sample names differ between the data matrix and the annotation table — the two files must share identical sample identifiers.

## Inputs

- Numerical lipid-by-sample matrix CSV (lipid names in first column, sample names in first row)
- Skyline results export as CSV
- Metabolomics Workbench study files (e.g. study ST001111)
- Sample annotation/clinical table CSV with sample names in the first column matching the data file

## Outputs

- LipidomicsExperiment object (extends SummarizedExperiment) with inferred lipid class/chain annotations
- QC plots (PCA score plots, box plots) and %CV-filtered lipid tables
- Normalized intensity tables (with or without internal standards)
- Volcano plots and differential analysis results by clinical variable
- OPLS-DA model with scores/loadings plots and ranked top lipids with class annotations
- Lipid set enrichment analysis (LSEA) results and visualization plots

## How to apply

Build a LipidomicsExperiment by importing either a numerical matrix (lipids as rows, samples as columns) via as_lipidomics_experiment, or a Skyline CSV export via read_skyline, then attach a sample-annotation CSV with add_sample_annotation, ensuring sample names are identical in both files. lipidr automatically infers lipid class and chain information from lipid names, so feature IDs must be parseable lipid nomenclature. For quality control, generate PCA score plots and box plots and filter lipids by %CV; normalize with or without internal standards before analysis. Run univariate analysis on loaded clinical variables (visualized as volcano plots), or multivariate analysis with mva(method='OPLS-DA', group_col=..., groups=...) restricting to the groups of interest, and rank features by loading magnitude with plot_mva_loadings; retrieve annotated top features with top_lipids (e.g. top.n=10). For class-level interpretation, apply the novel lipid set enrichment analysis (LSEA) to detect preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns.

## Related tools

- **lipidr** (Core R/Bioconductor package implementing the whole workflow: import, annotation, QC, normalization, univariate/multivariate analysis, and LSEA) — github.com/ahmohamed/lipidr
- **Skyline** (Source of targeted lipidomics CSV exports that lipidr reads as input)
- **Metabolomics Workbench API** (Used by lipidr to search, download, and reanalyze public lipidomics datasets such as ST001111)
- **limma** (Referenced for users building more complex design matrices for differential analysis)

## Examples

```
d <- as_lipidomics_experiment(read.csv("data_matrix.csv")); d <- add_sample_annotation(d, "data_clin.csv")
```

## Evaluation signals

- Sample names in the data file and the annotation CSV are identical, and add_sample_annotation completes without orphan samples.
- The LipidomicsExperiment carries inferred lipid class and chain annotations parsed from lipid names.
- For a two-class OPLS-DA, the mva call is restricted to the specified groups (e.g. Benign and Cancer) and top_lipids returns exactly the requested number of rows (e.g. top.n=10), each with a lipid name and class annotation, ordered by loading ranking.
- QC outputs (PCA score plots, box plots) are generated and %CV-based lipid filtering is documented before normalization.
- Differential results are visualizable as volcano plots against the loaded clinical variables.

## Limitations

- Lipid class and chain inference depends entirely on the lipid names provided; unparseable names yield missing annotations.
- Sample names must match exactly across the data file and the annotation/clinical table.
- The primary documented input path is targeted lipidomics (Skyline exports) or preprocessed numerical matrices, not raw instrument files.
- Multivariate models like OPLS-DA are supervised and must be restricted to the groups of interest; confounding-variable adjustment is described only for the univariate path.

## Evidence

- [readme] lipidr implements PCA, PCoA and OPLS(DA) to reveal patterns in data and discover variables related to an outcome of interest.: [evidence span withheld — anchor status: external_doc]
- [readme] Note the sample names must be identical in the two files.: [evidence span withheld — anchor status: external_doc]
