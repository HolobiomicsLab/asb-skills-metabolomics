---
name: qc-sample-scoreplot-visualization
description: Use when a metabonomic dataset (e.g. plasma 1H NMR spectra) contains
  repeated QC samples alongside experimental samples and you need to demonstrate analytical
  reproducibility or detect batch effects via PCA score plots.
when_to_use_negative:
- The dataset has no repeated QC/pooled-sample injections, so reproducibility cannot
  be assessed from score plots
- The goal is metabolite-phenotype association testing (MWAS with confounder adjustment),
  not analytical quality control
- The input is already a processed feature/association table rather than raw or bucketed
  spectral intensities of samples and QCs
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MWASTools
  role: Provides QC_PCA for computing principal-component scores and QC_PCA_scoreplot
    for drawing score plots with highlighted QC samples and the Hotelling ellipse
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment for executing the MWASTools QC pipeline (version >= 3.4)
- name: knitr
  role: Vignette engine used to render the QC PCA workflow and write figures to the
    fig.path output directory
- name: rmarkdown
  role: Vignette engine (knitr::rmarkdown) used with BiocStyle to document the QC
    scoreplot workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:efd16e4182cd16ee327604da5390a04b3b40c79fb71e4eac1d95914f147e0ee0
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# qc-sample-scoreplot-visualization

## Summary

Visualize principal-component score plots of 1H NMR metabonomic data with pooled QC samples highlighted and enclosed in a Hotelling confidence ellipse, to confirm analytical reproducibility and absence of batch effects. Implemented in MWASTools via QC_PCA and QC_PCA_scoreplot.

## When to use

Use when a metabonomic dataset (e.g. plasma 1H NMR spectra) contains repeated QC samples alongside experimental samples and you need to demonstrate analytical reproducibility or detect batch effects via PCA score plots.

## When NOT to use

- The dataset has no repeated QC/pooled-sample injections, so reproducibility cannot be assessed from score plots
- The goal is metabolite-phenotype association testing (MWAS with confounder adjustment), not analytical quality control
- The input is already a processed feature/association table rather than raw or bucketed spectral intensities of samples and QCs

## Inputs

- SummarizedExperiment object of 1H NMR metabonomic data containing experimental samples plus repeated pooled QC samples (e.g. metabo_SE with 506 experimental + 10 QC samples, δ 0.80–1.60 ppm)
- Vector/annotation identifying which rows are QC samples

## Outputs

- PCA score plots (PC1 vs PC2 and PC3 vs PC4) with QC samples highlighted and Hotelling confidence ellipse drawn (e.g. figures/qc_pca_scoreplot.png)
- Structured record (data frame/CSV) of PC1–PC4 scores for QC and experimental samples

## How to apply

Load the SummarizedExperiment containing both experimental samples and pooled QC injections (e.g. metabo_SE: 506 experimental + 10 QC samples over δ 0.80–1.60 ppm) with data('metabo_SE'). Run QC_PCA on the intensity matrix with scale=FALSE and center=TRUE to obtain sample principal-component scores. Call QC_PCA_scoreplot twice to draw the PC1-vs-PC2 and PC3-vs-PC4 score plots, highlighting QC samples and drawing the Hotelling confidence ellipse. Write the figures to the output path (fig.path='figures/') as qc_pca_scoreplot.png, and extract the PC1–PC4 scores of the QC samples into a structured record (data frame/CSV) alongside experimental samples for downstream inspection. The rationale: QC samples are repeated injections of a pooled specimen, so they should cluster tightly and centrally if the analytical run is reproducible; drift or spread indicates batch effects.

## Related tools

- **MWASTools** (Provides QC_PCA for computing principal-component scores and QC_PCA_scoreplot for drawing score plots with highlighted QC samples and the Hotelling ellipse) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment for executing the MWASTools QC pipeline (version >= 3.4))
- **knitr** (Vignette engine used to render the QC PCA workflow and write figures to the fig.path output directory)
- **rmarkdown** (Vignette engine (knitr::rmarkdown) used with BiocStyle to document the QC scoreplot workflow)

## Examples

```
data('metabo_SE'); pca <- QC_PCA(metabo_SE, scale = FALSE, center = TRUE); QC_PCA_scoreplot(pca, metinfo = NULL, ellip = TRUE)
```

## Evaluation signals

- QC samples appear tightly clustered at the center of the Hotelling ellipse in both PC1-vs-PC2 and PC3-vs-PC4 score plots
- No separation of QC samples into distinct groups, indicating absence of batch effects
- Figure file (e.g. qc_pca_scoreplot.png) is written to the configured fig.path='figures/' output directory
- Extracted PC1–PC4 score table contains scores for all 10 QC samples plus experimental samples and is structurally consistent with the input SummarizedExperiment
- PCA is run with the documented parameters scale=FALSE and center=TRUE

## Limitations

- Tight QC clustering in a PCA score plot supports absence of batch effects but does not quantify batch correction or per-metabolite drift
- Assessment depends on including repeated QC injections in the run; the example dataset uses only 10 QC samples
- Plot covers only PC1–PC4; systematic effects appearing in higher components will be missed

## Evidence

- [results] PCA score plots of the metabo_SE dataset (PC1 vs PC2 and PC3 vs PC4) show QC samples tightly clustered at the center of the Hotelling ellipse, confirming absence of batch effects and analytical reproducibility.: [evidence span withheld — anchor status: not_found]
- [abstract] MWASTools provides QC analysis as a key functionality of its integrated metabonomic pipeline.: "Key functionalities of our package include: quality control analysis"
- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies.: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [methods] The vignette engine and output configuration are defined in the package methods.: "%\VignetteEngine{knitr::rmarkdown}"
