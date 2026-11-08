---
name: metabolomics-preprocessing-pipeline-benchmarking
description: Use when comparing missing-value imputation and normalization pipelines
  for MS-Metabolomics replicate data, or when assessing whether technical vs biological
  replicate layers differ in metabolite reproducibility via MaRR.
when_to_use_negative:
- The goal is a comprehensive evaluation or recommendation of pre-processing steps
  — the authors state 'our goal was not to do a comprehensive evaluation of pre-processing
  steps, but instead to show how this measure can be used as a benchmark for comparison'
- The data lack technical or biological replicate structure (e.g., a single profile
  per condition), since MaRR reproducibility is estimated across replicate sample
  pairs
- The dataset is small/low-dimensional, since MaRR was designed for high-dimensional
  biological replicate experiments
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: marr
  role: Bioconductor package implementing the Maximum Rank Reproducibility (MaRR)
    maximal rank statistic used to score each pre-processing pipeline; Marr() is called
    per pipeline with FDR alpha=0.01
  repo: http://bioconductor.org/packages/marr
- name: MSPrep
  role: Provides the pre-processing (imputation/normalization) functionality benchmarked;
    installed from GitHub with ref='feature-generalize'
  repo: github.com/KechrisLab/MSPrep
- name: BiocManager
  role: Installer used to install the stable marr release from Bioconductor (requires
    R >= 4.0)
- name: xtable
  role: Generates the LaTeX code for the reproducibility summary tables in Table 1
    format
- name: devtools
  role: Used to install MSPrep (and the development version of marr) from GitHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:34e18100a4b5fca69cd31f8ea0073796ce7df04c979ff695fb7e6d27b2a6a925
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - MSPrep
  - R
  - devtools
  - marr
  - xtable
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# metabolomics-preprocessing-pipeline-benchmarking

## Summary

Benchmarks MS-Metabolomics pre-processing pipelines (imputation: BPCA/kNN; normalization: quantile/median/RUV) by applying the nonparametric MaRR maximal-rank reproducibility procedure to technical and biological replicate layers of the COPDGene Tech, BioTech, and Bio datasets and quantifying reproducible metabolites per sample pair and sample pairs per metabolite.

## When to use

Use when comparing missing-value imputation and normalization pipelines for MS-Metabolomics replicate data, or when assessing whether technical vs biological replicate layers differ in metabolite reproducibility via MaRR.

## When NOT to use

- The goal is a comprehensive evaluation or recommendation of pre-processing steps — the authors state 'our goal was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison'
- The data lack technical or biological replicate structure (e.g., a single profile per condition), since MaRR reproducibility is estimated across replicate sample pairs
- The dataset is small/low-dimensional, since MaRR was designed for high-dimensional biological replicate experiments

## Inputs

- MS-Metabolomics replicate data objects (metabolites as rows, samples as columns) for the Tech, BioTech, and Bio COPDGene datasets from the Real_Data 'data' folder
- Benchmarking dataset files from Metabolomics Workbench (Project IDs PR000438 and PR000907)

## Outputs

- 18 saved R objects (*.Rdata) in the 'saved_objects' folder, one per dataset per pre-processing pipeline
- Histograms of marr reproducibility scores (via MarrPlotFeatures/MarrPlotSamplepairs)
- LaTeX Table 1-style summaries of % reproducible metabolites per sample pair at >70%, >80%, >90% cutoffs (via xtable)
- Summary PDF 'MaRR_real_results.pdf' aggregating all plots and tables

## How to apply

Download the benchmarking datasets from Metabolomics Workbench (BioTech: Project ID PR000438; Bio: Project ID PR000907) and load the Tech/BioTech/Bio data objects from the Real_Data 'data' folder in R (>=4.0). Install the marr package from Bioconductor (BiocManager::install('marr')), install MSPrep from GitHub (ref='feature-generalize'), and source 'source_MaRR.R' from the 'src' folder. Run each of the 6 pre-processing pipelines (BPCA or kNN imputation combined with quantile, median, or RUV normalization) via the 18 R_Scripts, saving intermediate objects (*.Rdata) to 'saved_objects'. For each pipeline, call the Marr() maximal-rank statistic to estimate reproducibility of features per sample pair and sample pairs per feature, declaring reproducible signals at the conservative FDR level alpha=0.01 with data-dependent thresholds c_s and c_m chosen from histograms of the reproducibility scores. Summarize percentages of reproducible metabolites per sample pair above the >70%, >80%, and >90% cutoffs for technical and biological replicate layers, and generate LaTeX tables with xtable assembled into the 'MaRR_real_results.pdf' output.

## Related tools

- **marr** (Bioconductor package implementing the Maximum Rank Reproducibility (MaRR) maximal rank statistic used to score each pre-processing pipeline; Marr() is called per pipeline with FDR alpha=0.01) — http://bioconductor.org/packages/marr
- **MSPrep** (Provides the pre-processing (imputation/normalization) functionality benchmarked; installed from GitHub with ref='feature-generalize') — github.com/KechrisLab/MSPrep
- **BiocManager** (Installer used to install the stable marr release from Bioconductor (requires R >= 4.0))
- **xtable** (Generates the LaTeX code for the reproducibility summary tables in Table 1 format)
- **devtools** (Used to install MSPrep (and the development version of marr) from GitHub)

## Examples

```
BiocManager::install("marr"); MarrOutput <- Marr(object = dataSE, pSamplepairs = 0.75, pFeatures = 0.75, alpha = 0.01)
```

## Evaluation signals

- BioTech dataset reproduced percentages match Table 1: 90.08% of sample pairs have >70% reproducible metabolites for technical replicates vs 49.85% for biological replicates
- 18 saved R objects exist in 'saved_objects' (6 pre-processing pipelines x 3 datasets), matching the 18 R scripts in 'R_Scripts'
- Declared reproducible signals are controlled at FDR alpha=0.01, with c_s and c_m thresholds justified by histogram visualizations of marr scores
- Generated xtable LaTeX output reproduces the tables in the same order as shown in 'MaRR_real_results.pdf' (Output folder)
- Technical replicate layers show higher reproducibility than biological replicate layers across all three datasets (Tech, BioTech, Bio)

## Limitations

- True signals may fall in undetectable regions because of overlap between the reproducible and irreproducible components (lack of separation limits detection)
- MaRR has lower power when the proportion of reproducible metabolites is small
- The comparison is a benchmark demonstration, not a comprehensive evaluation of pre-processing steps

## Evidence

- [methods] Imputation- BPCA and kNN and Normalization- quantile, median and RUV: "Imputation- BPCA and kNN and Normalization- quantile, median and RUV"
- [discussion] MaRR for pipeline evaluation: "the MaRR procedure can also be employed to evaluate different pre-processing pipelines (e.g., missing value imputation, normalization)."
- [readme] 18 R scripts structure: "In this folder, there are 18 R scripts, i.e., 6 (6 pre-processing steps) R scripts for each data set."
- [readme] Dataset project IDs: "Project ID: PR000438 (BioTech dataset)"
- [methods] Benchmarking datasets on Metabolomics Workbench: "Data files for the benchmarking datasets are available from [Metabolomics Workbench]  (https://www.metabolomicsworkbench.org)"
- [readme] MaRR nonparametric maximal rank statistic: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
