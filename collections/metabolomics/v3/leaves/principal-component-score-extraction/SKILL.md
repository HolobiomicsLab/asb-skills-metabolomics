---
name: principal-component-score-extraction
description: Use when the analysis goal is quality control of a 1H NMR plasma metabonomic
  dataset — e.g. detecting batch effects or confirming run reproducibility by inspecting
  PCA score plots of QC samples.
when_to_use_negative:
- The data are already a metabolite feature table or a univariate association result
  — PCA-score QC is intended for the raw spectral matrix at the QC-analysis stage
  of the MWAS pipeline.
- You need metabolite–phenotype association models adjusted for epidemiological confounders
  (e.g. age, gender); MWASTools provides those separately and PCA score extraction
  is not a substitute.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: MWASTools
  role: Provides QC_PCA to compute PCA on the metabo_SE SummarizedExperiment and QC_PCA_scoreplot
    to render the PC1-vs-PC2 and PC3-vs-PC4 score plots with Hotelling ellipse; installed
    from Bioconductor.
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime for MWASTools; requires version >= 3.4 to execute the QC PCA workflow.
  repo: https://bioconductor.org/packages/MWASTools/
- name: knitr
  role: Vignette engine that sets fig.path='figures/' so the QC PCA score plots are
    written to the figures directory.
- name: rmarkdown
  role: Vignette engine (knitr::rmarkdown) that renders the MWASTools QC workflow
    documentation.
- name: BiocStyle
  role: HTML document style used for the MWASTools vignette.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:30404ceb4b17e751dfc89f27ec5b137ea82bd66b194a1bea8b4ae0fb73c90f6c
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
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# principal-component-score-extraction

## Summary

Use MWASTools to run PCA on a 1H NMR metabonomic SummarizedExperiment and extract sample PC scores, so that QC samples can be inspected for clustering within a Hotelling confidence ellipse as evidence of analytical reproducibility.

## When to use

Use when the analysis goal is quality control of a 1H NMR plasma metabonomic dataset — e.g. detecting batch effects or confirming run reproducibility by inspecting PCA score plots of QC samples.

## When NOT to use

- The data are already a metabolite feature table or a univariate association result — PCA-score QC is intended for the raw spectral matrix at the QC-analysis stage of the MWAS pipeline.
- You need metabolite–phenotype association models adjusted for epidemiological confounders (e.g. age, gender); MWASTools provides those separately and PCA score extraction is not a substitute.

## Inputs

- SummarizedExperiment object of 1H NMR metabonomic spectra (e.g. metabo_SE: 506 experimental + 10 QC samples, δ 0.80–1.60 ppm)

## Outputs

- PCA score plot of PC1 vs PC2 (with QC samples highlighted and Hotelling confidence ellipse) written to figures/qc_pca_scoreplot.png
- PCA score plot of PC3 vs PC4 (same styling) written to figures/
- Structured record (data frame/CSV) of PC1–PC4 sample scores for QC and experimental samples

## How to apply

Load the SummarizedExperiment shipped with MWASTools (metabo_SE, 506 experimental + 10 QC samples over δ 0.80–1.60 ppm) and call QC_PCA with scale=FALSE, center=TRUE to compute principal components from the NMR spectra. Then call QC_PCA_scoreplot twice — once for the PC1-vs-PC2 plane and once for the PC3-vs-PC4 plane — highlighting the 10 QC samples and drawing the Hotelling confidence ellipse. Write the resulting figures to the figures/ output path (fig.path='figures/' as set in the vignette chunk options) and export the PC1–PC4 scores of QC and experimental samples to a structured data frame or CSV for downstream inspection. Correct application is judged by the QC samples sitting tightly and centrally inside the Hotelling ellipse in both score plots, which indicates absence of batch effects and analytical reproducibility of the NMR data.

## Related tools

- **MWASTools** (Provides QC_PCA to compute PCA on the metabo_SE SummarizedExperiment and QC_PCA_scoreplot to render the PC1-vs-PC2 and PC3-vs-PC4 score plots with Hotelling ellipse; installed from Bioconductor.) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime for MWASTools; requires version >= 3.4 to execute the QC PCA workflow.) — https://bioconductor.org/packages/MWASTools/
- **knitr** (Vignette engine that sets fig.path='figures/' so the QC PCA score plots are written to the figures directory.)
- **rmarkdown** (Vignette engine (knitr::rmarkdown) that renders the MWASTools QC workflow documentation.)
- **BiocStyle** (HTML document style used for the MWASTools vignette.)

## Examples

```
Rscript -e "library(MWASTools); data('metabo_SE'); pca <- QC_PCA(metabo_SE, scale=FALSE, center=TRUE); QC_PCA_scoreplot(pca, PCx='PC1', PCy='PC2'); QC_PCA_scoreplot(pca, PCx='PC3', PCy='PC4')"
```

## Evaluation signals

- PC1–PC4 scores are returned for all 506 experimental and 10 QC samples without missing values.
- QC_PCA is called with scale=FALSE and center=TRUE; any deviation from these parameters is documented.
- Both score plots (PC1 vs PC2 and PC3 vs PC4) are produced and saved to figures/qc_pca_scoreplot.png.
- In each score plot the 10 QC samples fall within the Hotelling confidence ellipse and are tightly clustered near its center.
- The exported sample-score table retains sample identity labels so QC vs experimental samples can be distinguished.

## Limitations

- The skill is specific to QC of 1H NMR metabonomic data as provided in MWASTools' example dataset (metabo_SE); other spectral formats or parameterisations (e.g. scaling) are not covered by the vignette workflow.
- Absence of batch effects is inferred visually from the Hotelling ellipse; the vignette does not supply a quantitative threshold or statistical test for reproducibility.
- The vignette uses only the δ 0.80–1.60 ppm spectral region, so QC conclusions are limited to that chemical-shift window.

## Evidence

- [abstract] Key functionalities of our package include: quality control analysis: "Key functionalities of our package include: quality control analysis"
- [methods] VignetteIndexEntry{MWASTools}: "VignetteIndexEntry{MWASTools}"
- [abstract] The MWASTools R package is implemented in R (version > =3.4): "The MWASTools R package is implemented in R (version > =3.4)"
- [other] PCA score plots of the metabo_SE dataset (PC1 vs PC2 and PC3 vs PC4) show QC samples tightly clustered at the center of the Hotelling ellipse: [evidence span withheld — anchor status: not_found]
- [other] Load the metabo_SE SummarizedExperiment (506 experimental + 10 QC samples, δ 0.80–1.60 ppm) shipped with the MWASTools package via data('metabo_SE'): [evidence span withheld — anchor status: not_found]
