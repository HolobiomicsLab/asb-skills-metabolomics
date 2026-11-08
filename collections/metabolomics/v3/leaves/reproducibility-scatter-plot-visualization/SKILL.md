---
name: reproducibility-scatter-plot-visualization
description: Use when you have MaRR-classified reproducible/irreproducible metabolites
  and per-metabolite RSD values and need to show graphically whether a conventional
  CV/RSD cutoff (e.g., 25%) separates the two classes.
when_to_use_negative:
- When the goal is a comprehensive evaluation of pre-processing steps rather than
  a benchmark comparison — the paper states 'our goal was not to do a comprehensive
  evaluation of pre-processing steps, but instead to show how this measure can be
  used as a benchmark for comparison.'
- When reproducible and irreproducible components overlap heavily and fall in undetectable
  regions, making MaRR labels unreliable for the comparison.
- When data have not been pre-processed (e.g., missing value imputation and normalization
  such as BPCA/kNN imputation and quantile/median/RUV normalization) — MaRR is applied
  after data processing.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: marr
  role: R/Bioconductor package computing the Maximum Rank Reproducibility maximal
    rank statistic used to classify metabolites as reproducible/irreproducible before
    plotting against RSD
  repo: http://bioconductor.org/packages/marr
- name: R
  role: Analysis environment (R >= 4.0) in which MaRR functions are sourced and the
    scatter plots and tables are generated
  repo: https://github.com/Ghoshlab/marr_paper_evaluations
- name: xtable
  role: R package used to generate the LaTeX comparison table (Table S4) accompanying
    the reproducibility-vs-RSD scatter plots
- name: MSPrep
  role: Pre-processing pipeline (installed from GitHub, ref='feature-generalize')
    used to prepare metabolomics data (e.g., RF-quantile imputation/normalization)
    prior to the reproducibility comparison
  repo: https://github.com/KechrisLab/MSPrep
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:f5f279de72718daf88a03f7c6b8486205518a6d42f40174ac7095934b0b4edd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSPrep
  - R
  - marr
  - xtable
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reproducibility-scatter-plot-visualization

## Summary

Generate scatter plots of MaRR reproducibility versus relative standard deviation (RSD) for biological and technical replicates, overlaying a 25% RSD filtering cutoff, to visually demonstrate whether conventional RSD filtering removes MaRR-identified irreproducible metabolites.

## When to use

Use when you have MaRR-classified reproducible/irreproducible metabolites and per-metabolite RSD values and need to show graphically whether a conventional CV/RSD cutoff (e.g., 25%) separates the two classes.

## When NOT to use

- When the goal is a comprehensive evaluation of pre-processing steps rather than a benchmark comparison — the paper states 'our goal was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison.'
- When reproducible and irreproducible components overlap heavily and fall in undetectable regions, making MaRR labels unreliable for the comparison.
- When data have not been pre-processed (e.g., missing value imputation and normalization such as BPCA/kNN imputation and quantile/median/RUV normalization) — MaRR is applied after data processing.

## Inputs

- RF-quantile pre-processed BioTech dataset R objects from the 'saved_objects' folder of the Real_Data repository
- MaRR reproducibility outputs (percent reproducible sample pairs per feature / features per sample pair) computed with the marr package at pSamplepairs=0.75 and pFeatures=0.75
- Raw data from Metabolomics Workbench Project ID: PR000438 (BioTech dataset)

## Outputs

- Scatter plots of MaRR reproducibility vs RSD for biological and technical replicate layers (reproducing Additional file 1: Figs. S24–S25) with a 25% RSD cutoff annotated
- LaTeX comparison table (reproducing Table S4) generated with xtable
- Plots and tables saved to the 'Output' folder as summarized in MaRR_real_results.pdf

## How to apply

Compute RSD of metabolite abundances across biological replicates and across technical replicates for every metabolite in the RF-quantile pre-processed dataset, then scatter-plot MaRR reproducibility (from the maximal rank statistic with a 75% sample-pair reproducibility threshold) against RSD for each replicate layer, reproducing Additional file 1: Figs. S24–S25. Overlay/annotate a 25% RSD cutoff line to reveal that it fails to remove the MaRR-labeled irreproducible metabolites. Generate the accompanying LaTeX comparison table with the xtable package in the same order as the tables of MaRR_real_results.pdf. Plot and table scripts belong in the 'Plot_R_Scripts' and 'Table_R_Scripts' folders and read the saved R objects in 'saved_objects', writing summarized outputs to the 'Output' folder.

## Related tools

- **marr** (R/Bioconductor package computing the Maximum Rank Reproducibility maximal rank statistic used to classify metabolites as reproducible/irreproducible before plotting against RSD) — http://bioconductor.org/packages/marr
- **R** (Analysis environment (R >= 4.0) in which MaRR functions are sourced and the scatter plots and tables are generated) — https://github.com/Ghoshlab/marr_paper_evaluations
- **xtable** (R package used to generate the LaTeX comparison table (Table S4) accompanying the reproducibility-vs-RSD scatter plots)
- **MSPrep** (Pre-processing pipeline (installed from GitHub, ref='feature-generalize') used to prepare metabolomics data (e.g., RF-quantile imputation/normalization) prior to the reproducibility comparison) — https://github.com/KechrisLab/MSPrep

## Examples

```
library(marr); MarrOutput <- Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.05); plot(MarrFeaturesfiltered(MarrOutput) ~ rsd, abline(v=0.25))
```

## Evaluation signals

- Reproduced scatter plots must match the qualitative pattern of Additional file 1: Figs. S24–S25
- Table S4 must confirm that a 25% RSD cutoff does not remove any of the irreproducible metabolites identified by MaRR in the BioTech dataset
- MaRR classification must use the maximal rank statistic with a 75% sample-pair reproducibility threshold applied separately to biological and technical replicate layers
- Output tables must be ordered identically to the tables in MaRR_real_results.pdf in the 'Output' folder

## Limitations

- A 25% RSD filtering cutoff does not remove any MaRR-identified irreproducible metabolites in the BioTech dataset, so RSD fails to separate reproducible from irreproducible metabolites
- Real signals might fall in undetectable regions because of overlapping between the reproducible and irreproducible components
- Results depend on the chosen pre-processing pipeline (imputation: BPCA/kNN; normalization: quantile/median/RUV); only six pre-processing steps were benchmarked

## Evidence

- [methods] task workflow for RSD vs MaRR comparison: [evidence span withheld — anchor status: not_found]
- [results] cutoff annotation finding: [evidence span withheld — anchor status: not_found]
- [methods] table generation method: "These R scripts also generate the latex code using the package "xtable""
- [abstract] pre-processing prerequisite: "we applied the MaRR procedure after data processing to explore reproducibility across technical or biological samples"
- [intro] RSD as conventional measure: "The most common approach is calculating the coefficient of variation (CV) across replicates."
- [readme] threshold choice via visualization: "our approach also allows the users to select optimal filtering threshold values for the identification of reproducible features and sample pairs based on output visualization checks (histograms)"
