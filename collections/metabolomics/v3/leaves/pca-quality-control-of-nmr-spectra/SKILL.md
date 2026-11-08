---
name: pca-quality-control-of-nmr-spectra
description: Use when QC of a large-scale 1H NMR metabonomic dataset (e.g. plasma
  spectra with pooled QC replicates) is required before metabolome-wide association
  analysis, to verify analytical reproducibility and absence of batch effects.
when_to_use_negative:
- The dataset contains no pooled QC replicate samples, so no reproducibility check
  via QC clustering is possible
- The goal is metabolite-phenotype association modelling (partial correlations or
  generalized linear models adjusted for confounders such as age or gender) rather
  than analytical quality control
- Input is already a PCA result or a feature table rather than binned NMR spectral
  data in a SummarizedExperiment
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MWASTools
  role: Provides QC_PCA and QC_PCA_scoreplot functions plus the metabo_SE example
    SummarizedExperiment used to run the PCA-based QC
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which the MWASTools QC-PCA workflow is executed (version
    >= 3.4)
  repo: github.com/AndreaRMICL/MWASTools
- name: knitr
  role: Vignette engine (knitr::rmarkdown) that renders the documented QC-PCA workflow
    and writes figures to the figures/ path
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:3f44553a0b84cb0f51ab9471e441585483af9f13779a79bf5b7aa308c4453847
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - MWASTools
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# pca-quality-control-of-nmr-spectra

## Summary

A quality-control step from MWASTools that assesses analytical reproducibility of plasma 1H NMR metabonomic runs by checking whether pooled QC samples cluster tightly and centrally in PCA score plots (PC1-vs-PC2 and PC3-vs-PC4) inside a Hotelling confidence ellipse, confirming absence of batch effects.

## When to use

Use when QC of a large-scale 1H NMR metabonomic dataset (e.g. plasma spectra with pooled QC replicates) is required before metabolome-wide association analysis, to verify analytical reproducibility and absence of batch effects.

## When NOT to use

- The dataset contains no pooled QC replicate samples, so no reproducibility check via QC clustering is possible
- The goal is metabolite-phenotype association modelling (partial correlations or generalized linear models adjusted for confounders such as age or gender) rather than analytical quality control
- Input is already a PCA result or a feature table rather than binned NMR spectral data in a SummarizedExperiment

## Inputs

- SummarizedExperiment of binned plasma 1H NMR spectra (e.g. metabo_SE: 506 experimental + 10 QC samples, δ 0.80–1.60 ppm)
- Sample-class annotations distinguishing QC replicate samples from experimental samples

## Outputs

- PCA score plots (PC1-vs-PC2 and PC3-vs-PC4) with QC samples highlighted and Hotelling ellipse, e.g. figures/qc_pca_scoreplot.png
- Structured table (data frame/CSV) of PC1–PC4 scores for QC and experimental samples

## How to apply

Load the bundled metabo_SE SummarizedExperiment (506 experimental samples + 10 QC samples binned over δ 0.80–1.60 ppm) shipped with MWASTools via data('metabo_SE'), or supply an equivalently structured SummarizedExperiment of binned NMR spectra with class labels marking QC samples. Run QC_PCA on the dataset with scale=FALSE and center=TRUE to obtain sample principal-component scores, matching the vignette settings. Call QC_PCA_scoreplot twice to draw the PC1-vs-PC2 and PC3-vs-PC4 score plots with QC samples highlighted and the Hotelling confidence ellipse drawn, and write the figures to the figures/ output path as qc_pca_scoreplot.png. Finally extract the PC1–PC4 scores of the 10 QC samples (alongside experimental samples) into a structured data frame/CSV for downstream inspection. The rationale is that tightly, centrally clustered QC samples within the Hotelling ellipse indicate good analytical reproducibility of the NMR run, a QC stage that MWASTools provides as the first key functionality of its MWAS pipeline.

## Related tools

- **MWASTools** (Provides QC_PCA and QC_PCA_scoreplot functions plus the metabo_SE example SummarizedExperiment used to run the PCA-based QC) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which the MWASTools QC-PCA workflow is executed (version >= 3.4)) — github.com/AndreaRMICL/MWASTools
- **knitr** (Vignette engine (knitr::rmarkdown) that renders the documented QC-PCA workflow and writes figures to the figures/ path)

## Examples

```
R> data("metabo_SE"); R> pca_res <- QC_PCA(metabo_SE, scale = FALSE, center = TRUE); R> QC_PCA_scoreplot(pca_res, PCs = c(1, 2))
```

## Evaluation signals

- QC samples appear tightly and centrally clustered within the Hotelling confidence ellipse in both PC1-vs-PC2 and PC3-vs-PC4 score plots
- No batch-effect separation is visible between QC and experimental samples across the first four principal components
- PC1–PC4 scores are extracted for all 10 QC samples (plus experimental samples) into a structured CSV/data frame
- QC_PCA was run with scale=FALSE and center=TRUE, matching the vignette parameters
- Score-plot figures are written to the figures/ output path (e.g. qc_pca_scoreplot.png)

## Limitations

- Central QC clustering demonstrates analytical reproducibility of the NMR run only; it does not validate downstream metabolite-phenotype associations or account for epidemiological confounders (e.g. age, gender)
- The documented example relies on the bundled metabo_SE dataset (δ 0.80–1.60 ppm region, 10 QC samples); other spectral regions or QC designs may need re-parameterisation
- Interpretation depends on the Hotelling ellipse drawn by QC_PCA_scoreplot; conclusions are visual rather than quantitative unless PC scores are exported and inspected

## Evidence

- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [abstract] Key functionalities of our package include: quality control analysis: "Key functionalities of our package include: quality control analysis"
- [intro] Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis"
- [supplementary] Assessment of analytical reproducibility of 1H NMR spectroscopy based metabonomics for large-scale epidemiological research: the INTERMAP study: "Assessment of analytical reproducibility of 1H NMR spectroscopy based metabonomics for large-scale epidemiological research: the INTERMAP study"
