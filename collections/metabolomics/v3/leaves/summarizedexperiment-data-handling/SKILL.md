---
name: summarizedexperiment-data-handling
description: Use when metabolomics/1H-NMR data must be loaded into R as a SummarizedExperiment
  (metabo_SE) so MWASTools functions can operate on feature intensities and clinical
  metadata together.
when_to_use_negative:
- Data is a plain feature table (CSV/Excel of metabolites x samples) that has not
  been assembled into a SummarizedExperiment with sample metadata — convert/containerize
  it first
- The analysis target is proteomic/genomic rather than metabonomic (1H-NMR) data,
  since MWASTools functions like QC_CV_specNMR and MWAS_skylineNMR are NMR-specific
- The user needs a multivariate modeling workflow; the paper's rationale is univariate
  MWAS with explicit confounder handling (partial correlations, GLMs) instead
tools:
- name: MWASTools
  role: Supplies the SummarizedExperiment-based data container (example metabo_SE)
    and all pipeline functions (QC_CV, MWAS_stats, MWAS_skylineNMR, STOCSY_NMR) that
    consume it
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment (version >= 3.4) in which the SummarizedExperiment object
    is loaded and manipulated
- name: knitr
  role: Vignette engine used to render the MWASTools vignette workflows that operate
    on metabo_SE
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:dfc3ecb8c23ce33fa594e0af3f1cf0ead7e4dc2fba28ca1f3a7d3382e7c84e68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  - knitr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# summarizedexperiment-data-handling

## Summary

Load and manipulate metabonomic data as SummarizedExperiment objects (metabo_SE) in R/Bioconductor via MWASTools, so that QC (QC_CV, QC_CV_specNMR), MWAS (MWAS_stats), visualization (MWAS_skylineNMR) and STOCSY (STOCSY_NMR) all operate on one shared assay-plus-metadata container.

## When to use

Use when metabolomics/1H-NMR data must be loaded into R as a SummarizedExperiment (metabo_SE) so MWASTools functions can operate on feature intensities and clinical metadata together.

## When NOT to use

- Data is a plain feature table (CSV/Excel of metabolites x samples) that has not been assembled into a SummarizedExperiment with sample metadata — convert/containerize it first
- The analysis target is proteomic/genomic rather than metabonomic (1H-NMR) data, since MWASTools functions like QC_CV_specNMR and MWAS_skylineNMR are NMR-specific
- The user needs a multivariate modeling workflow; the paper's rationale is univariate MWAS with explicit confounder handling (partial correlations, GLMs) instead

## Inputs

- SummarizedExperiment object (metabo_SE) with a metabolic-feature assay matrix and sample metadata
- Clinical/phenotype metadata columns such as BMI, Age, Gender, T2D stored in the object's colData
- QC replicate sample identifiers (e.g. 'QC1') for reproducibility checks

## Outputs

- A SummarizedExperiment object usable by MWASTools QC, MWAS, visualization and STOCSY functions
- Per-feature CV table (exported as cv_table.csv) with one CV value per metabolic feature
- 3-column MWAS result matrix (correlation estimate, raw p-value, BH-adjusted pFDR) exported as MWAS_BMI_matrix.csv
- Skyline plot figure (e.g. MWAS_skyline_BMI.png) indexed by 1H-NMR chemical shift in ppm

## How to apply

Represent the metabonomic dataset as a SummarizedExperiment object (e.g. the packaged example metabo_SE in MWASTools) where the assay holds metabolic feature intensities across samples (including QC replicates such as 'QC1') and colData holds clinical metadata (BMI, Age, Gender, T2D). Load it with library(MWASTools) and data(metabo_SE), then pass this single object to the pipeline functions: QC_CV(metabo_SE, plot_hist=FALSE) for per-feature CVs across QC samples, QC_CV_specNMR(metabo_SE, ref_sample="QC1") for spectral reproducibility, MWAS_stats(disease_id="BMI", confounder_ids=c("Age","Gender","T2D"), assoc_method="spearman", mt_method="BH") for confounder-adjusted MWAS, MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample="QC1") for skyline plots, and STOCSY_NMR(metabo_SE, ppm_query=1.04) for metabolite assignment. The rationale is that the SummarizedExperiment container keeps features, QC replicate labels and phenotype confounders in one object, so the confounder-aware univariate MWAS pipeline (rather than multivariate models that ignore age/gender) can be run reproducibly. Verify correctness by checking that the object retains one row per metabolic feature and that downstream outputs (e.g. cv_table.csv, MWAS_BMI_matrix.csv with 3 columns) derive from the same container.

## Related tools

- **MWASTools** (Supplies the SummarizedExperiment-based data container (example metabo_SE) and all pipeline functions (QC_CV, MWAS_stats, MWAS_skylineNMR, STOCSY_NMR) that consume it) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment (version >= 3.4) in which the SummarizedExperiment object is loaded and manipulated)
- **knitr** (Vignette engine used to render the MWASTools vignette workflows that operate on metabo_SE)

## Examples

```
library(MWASTools); data(metabo_SE); cv <- QC_CV(metabo_SE, plot_hist=FALSE)
```

## Evaluation signals

- The loaded object is a SummarizedExperiment with one row per metabolic feature and colData containing BMI, Age, Gender and T2D columns
- QC_CV on the object yields a CV value for every metabolic feature; exported cv_table.csv has no missing feature rows
- Fractions of features with CV < 0.30 and CV < 0.15 computed from the container reproduce ~99% and ~92% within rounding tolerance
- MWAS_stats output from the container is a 3-column matrix (estimate, raw p-value, BH-adjusted pFDR) with p-values within [0,1] and one row per feature
- MWAS_skylineNMR and STOCSY_NMR called on the same object reproduce the paper's skyline lipid region (δ 1.60–0.80 ppm) and the valine doublets at δ 1.04 / 0.99 ppm

## Limitations

- Multivariate models used by other metabonomic packages do not properly account for confounding factors such as age and gender; the SummarizedExperiment-based MWASTools workflow is univariate and confounder-aware, so results may differ from multivariate analyses
- Requires R version >= 3.4 and Bioconductor installation
- CV filtering at the 0.30 threshold is justified by the case-study reproducibility (~99% of features pass); datasets with poorer QC reproducibility may require different filtering decisions

## Evidence

- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [intro] Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis; metabolite-phenotype association models (partial correlations,: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis; metabolite-phenotype association models (partial correlations,"
- [intro] Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders.: "Here, we present a package to perform MWAS using univariate hypothesis testing with efficient handling of epidemiological confounders."
