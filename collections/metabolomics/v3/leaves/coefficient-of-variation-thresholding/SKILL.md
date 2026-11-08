---
name: coefficient-of-variation-thresholding
description: Use when a metabonomic feature matrix is accompanied by a per-feature
  CV table and non-reproducible spectral features must be removed before downstream
  MWAS analysis.
when_to_use_negative:
- The dataset has no quality-control replicates or CV table, so no reproducibility
  criterion can be computed — association-model steps of MWASTools are more appropriate.
- The feature matrix is not a SummarizedExperiment of metabonomic (e.g. 1H NMR) features;
  CV_filter expects the metabo_SE/metabo_CV structures.
- The goal is phenotype association testing or STOCSY metabolite assignment rather
  than QC filtering of features.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MWASTools
  role: Supplies CV_filter(), which takes metabo_SE and metabo_CV and applies the
    CV_th cut-off to return the filtered SummarizedExperiment
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment (R >= 3.4) in which the filtering call is executed and
    the filtered object is saved as RDS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:d5ac1d28373cd6b02182d36931fc35130e23d04ea767309b7ec432851ba167ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# coefficient-of-variation-thresholding

## Summary

Filters a metabonomic SummarizedExperiment so that only metabolomic features whose coefficient of variation (CV) across quality-control measurements is at or below a reproducibility cut-off (CV_th = 0.30) are retained. It is the reproducibility/QC filtering step of the MWASTools pipeline for 1H-NMR metabonomic data.

## When to use

Use when a metabonomic feature matrix is accompanied by a per-feature CV table and non-reproducible spectral features must be removed before downstream MWAS analysis.

## When NOT to use

- The dataset has no quality-control replicates or CV table, so no reproducibility criterion can be computed — association-model steps of MWASTools are more appropriate.
- The feature matrix is not a SummarizedExperiment of metabonomic (e.g. 1H NMR) features; CV_filter expects the metabo_SE/metabo_CV structures.
- The goal is phenotype association testing or STOCSY metabolite assignment rather than QC filtering of features.

## Inputs

- metabo_SE: SummarizedExperiment object containing the metabolomic feature matrix (features indexed by ppm)
- metabo_CV: per-feature coefficient-of-variation table produced by the upstream CV analysis step

## Outputs

- Filtered SummarizedExperiment (reassigned to metabo_SE) retaining only features with CV <= 0.30
- RDS file of the filtered SummarizedExperiment
- Log of retained feature count and retained features' ppm values

## How to apply

Load the SummarizedExperiment object (metabo_SE) holding the metabolomic feature matrix together with the per-feature CV table (metabo_CV) produced by the upstream CV analysis. Call MWASTools::CV_filter(metabo_SE, metabo_CV, CV_th = 0.30) so that only features with coefficient of variation <= 0.30 are kept in the metabolic matrix; the function returns the filtered SummarizedExperiment, typically reassigned to metabo_SE. Persist the filtered object as an RDS file and record the number of retained features and their ppm values so the filtering is auditable. The rationale, per the MWASTools paper, is that quality control analysis is a core pipeline stage for large-scale epidemiological metabonomic studies, where analytical reproducibility (e.g. as assessed in the INTERMAP 1H NMR study) determines which features are trustworthy for metabolome-wide association analysis.

## Related tools

- **MWASTools** (Supplies CV_filter(), which takes metabo_SE and metabo_CV and applies the CV_th cut-off to return the filtered SummarizedExperiment) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment (R >= 3.4) in which the filtering call is executed and the filtered object is saved as RDS)

## Examples

```
metabo_SE <- MWASTools::CV_filter(metabo_SE, metabo_CV, CV_th = 0.30)
```

## Evaluation signals

- Every retained feature's CV value in metabo_CV is <= 0.30.
- Every removed feature's CV value in metabo_CV exceeds 0.30.
- The number of retained features equals the count of entries in metabo_CV with CV <= 0.30.
- The filtered object is a valid SummarizedExperiment with the retained features' ppm values recorded alongside the RDS file.
- Re-running the filter on the same metabo_SE and metabo_CV with CV_th = 0.30 reproduces an identical feature set.

## Limitations

- The threshold value 0.30 is the cut-off used in this workflow; other reproducibility tolerances require re-running with a different CV_th.
- Filtering is only as good as the upstream CV table — features absent from metabo_CV cannot be judged reproducible or not.
- This step addresses analytical reproducibility only; the paper notes broader limitations of existing analysis software, e.g. failure to properly account for confounding factors such as age and gender, which are handled by separate association-model steps.

## Evidence

- [abstract] MWASTools provides an integrated pipeline for metabonomic data analysis including QC, MWAS, visualization, STOCSY metabolite assignment, and biological interpretation: "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models"
- [intro] MWASTools pipeline stage positioning of QC relative to association models: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis"
- [supplementary] Analytical reproducibility assessment underlying CV-based filtering in 1H NMR metabonomics: "Assessment of analytical reproducibility of 1H NMR spectroscopy based metabonomics for large-scale epidemiological research: the INTERMAP study"
