---
name: nmr-metabonomic-dataset-loading
description: Use when starting an MWASTools-based metabonomic analysis or vignette-style
  reproducibility task that needs a ready-made plasma 1H NMR dataset with annotated
  QC samples.
when_to_use_negative:
- The analysis already has its own NMR spectral matrix (e.g. user-supplied binned
  spectra) — this skill only supplies the package's bundled example dataset, not a
  general NMR importer
- The target data are MS-based metabolomics or non-NMR omics — MWASTools and metabo_SE
  are specific to 1H NMR metabonomic data in epidemiological studies
- A downstream step other than QC/PCA on metabo_SE is needed and a different MWASTools
  example object (e.g. the one for MWAS/STOCSY functions) is the appropriate bundled
  input
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MWASTools
  role: Bioconductor package that ships the metabo_SE SummarizedExperiment example
    dataset and provides the downstream QC_PCA / QC_PCA_scoreplot functions it feeds
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment (version >= 3.4) in which metabo_SE is loaded and inspected
    as a SummarizedExperiment
- name: knitr
  role: Vignette engine used to render the reproducible workflow that loads metabo_SE
    and writes figures to fig.path='figures/'
- name: rmarkdown
  role: Vignette rendering format (%\VignetteEngine{knitr::rmarkdown}) under which
    the dataset-loading and QC-PCA chunks execute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:55050d824f991a05ec47c9e6ee7727abb308c884c241a82c64aee0784126dfe7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - MWASTools
  - R
  - knitr
  - rmarkdown
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# nmr-metabonomic-dataset-loading

## Summary

Loads the metabo_SE SummarizedExperiment example dataset (506 experimental + 10 QC plasma 1H NMR samples, δ 0.80–1.60 ppm) shipped with the MWASTools Bioconductor package as the standard input for MWAS quality-control and association workflows.

## When to use

Use when starting an MWASTools-based metabonomic analysis or vignette-style reproducibility task that needs a ready-made plasma 1H NMR dataset with annotated QC samples.

## When NOT to use

- The analysis already has its own NMR spectral matrix (e.g. user-supplied binned spectra) — this skill only supplies the package's bundled example dataset, not a general NMR importer
- The target data are MS-based metabolomics or non-NMR omics — MWASTools and metabo_SE are specific to 1H NMR metabonomic data in epidemiological studies
- A downstream step other than QC/PCA on metabo_SE is needed and a different MWASTools example object (e.g. the one for MWAS/STOCSY functions) is the appropriate bundled input

## Inputs

- metabo_SE SummarizedExperiment object bundled with the MWASTools Bioconductor package (loaded via data('metabo_SE'))
- R session (version >= 3.4) with MWASTools installed from Bioconductor

## Outputs

- In-memory SummarizedExperiment with 516 samples (506 experimental + 10 QC) over the δ 0.80–1.60 ppm 1H NMR binning region
- Structured extraction of PC1–PC4 scores for the 10 QC samples (data frame/CSV) alongside experimental samples, as done in the task workflow

## How to apply

Load the dataset into R via data('metabo_SE') from MWASTools, which returns a SummarizedExperiment containing 506 experimental samples plus 10 QC samples restricted to the δ 0.80–1.60 ppm spectral region. Verify the object's dimensions (516 total samples) and that the QC flag column distinguishes the 10 QC from the 506 experimental samples before any downstream QC_PCA or MWAS call. Run downstream steps with QC_PCA using scale=FALSE and center=TRUE, as in the package vignette (rendered with knitr::rmarkdown and BiocStyle, fig.path='figures/'). The rationale is that MWASTools ships this SummarizedExperiment precisely so users can exercise the full QC → MWAS → visualization → STOCSY pipeline on canonical NMR data without external data retrieval.

## Related tools

- **MWASTools** (Bioconductor package that ships the metabo_SE SummarizedExperiment example dataset and provides the downstream QC_PCA / QC_PCA_scoreplot functions it feeds) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment (version >= 3.4) in which metabo_SE is loaded and inspected as a SummarizedExperiment)
- **knitr** (Vignette engine used to render the reproducible workflow that loads metabo_SE and writes figures to fig.path='figures/')
- **rmarkdown** (Vignette rendering format (%\VignetteEngine{knitr::rmarkdown}) under which the dataset-loading and QC-PCA chunks execute)

## Examples

```
library(MWASTools); data('metabo_SE'); qc_flag <- SummarizedExperiment::colData(metabo_SE)$QC
```

## Evaluation signals

- After loading, dim(metabo_SE) reports 516 samples: 506 experimental plus 10 QC samples
- The QC column identifies exactly 10 samples flagged as QC, matching the dataset shipped with MWASTools
- The spectral features fall within the δ 0.80–1.60 ppm region used for the metabo_SE dataset
- Downstream QC_PCA(scale=FALSE, center=TRUE) runs without error on the loaded object and PC1–PC4 scores can be extracted for all 516 samples
- Reproducibility check: QC samples cluster tightly at the center of the Hotelling ellipse in PC1-vs-PC2 and PC3-vs-PC4 score plots

## Limitations

- metabo_SE covers only the δ 0.80–1.60 ppm spectral region and 516 samples, so it cannot stand in for full-spectrum or externally supplied NMR data
- QC-sample counts (10) and sample counts (506) are fixed properties of the bundled object; loading does not generalize to other cohort sizes
- The dataset is a vignette example (INTERMAP-related NMR metabonomics context), so results reproduce the published QC behaviour rather than validating new data

## Evidence

- [other] Load the metabo_SE SummarizedExperiment (506 experimental + 10 QC samples, δ 0.80–1.60 ppm) shipped with the MWASTools package via data('metabo_SE').: [evidence span withheld — anchor status: not_found]
- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [abstract] The MWASTools R package is implemented in R (version > =3.4): "The MWASTools R package is implemented in R (version > =3.4)"
- [intro] quality control (QC) analysis; metabolite-phenotype association models (partial correlations, generalized linear models): "quality control (QC) analysis; metabolite-phenotype association models (partial correlations, generalized linear models)"
- [methods] %\VignetteEngine{knitr::rmarkdown}: "%\VignetteEngine{knitr::rmarkdown}"
