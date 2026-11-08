---
name: marr-reproducibility-analysis
description: Use when replicate MS-metabolomics abundance data (biological or technical
  replicates, e.g., RF-quantile pre-processed BioTech data from Metabolomics Workbench)
  must be assessed for feature reproducibility or when comparing data-driven reproducibility
  labels against CV/RSD filtering cutoffs.
when_to_use_negative:
- When the goal is a comprehensive evaluation of pre-processing steps — the authors
  state the goal was not comprehensive pre-processing evaluation, only benchmarking
  comparisons
- When the data are single-sample (no biological or technical replicate structure),
  since MaRR assesses reproducibility across replicate pairs
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: marr
  role: R/Bioconductor package implementing the MaRR maximal rank statistic to compute
    reproducibility of features per sample pair and sample pairs per feature, with
    filtering thresholds
  repo: http://bioconductor.org/packages/marr
- name: marr_paper_evaluations
  role: Repository of scripts and saved R objects (Real_Data folder, Plot_R_Scripts,
    Table_R_Scripts, src/source_MaRR.R) to reproduce the MaRR vs RSD comparison plots
    and tables
  repo: github.com/Ghoshlab/marr_paper_evaluations
- name: xtable
  role: Generates the LaTeX code for the comparison tables in the same order as MaRR_real_results.pdf
- name: MSPrep
  role: GitHub-installed pre-processing package for the metabolomics pipelines preceding
    MaRR application
  repo: KechrisLab/MSPrep
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:390b819c10869a63230789b9d564ce1a0012a1169a1ca492b2188742dfd90855
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSPrep
  - R
  - marr
  - marr_paper_evaluations
  - xtable
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# marr-reproducibility-analysis

## Summary

Applies the nonparametric Maximum Rank Reproducibility (MaRR) procedure via the R/Bioconductor `marr` package to classify metabolites as reproducible or irreproducible in MS-metabolomics replicate experiments, and benchmarks the resulting labels against conventional RSD/CV filtering (e.g., a 25% RSD cutoff) on datasets like the BioTech dataset (PR000438).

## When to use

Use when replicate MS-metabolomics abundance data (biological or technical replicates, e.g., RF-quantile pre-processed BioTech data from Metabolomics Workbench) must be assessed for feature reproducibility or when comparing data-driven reproducibility labels against CV/RSD filtering cutoffs.

## When NOT to use

- When the goal is a comprehensive evaluation of pre-processing steps — the authors state the goal was not comprehensive pre-processing evaluation, only benchmarking comparisons
- When the data are single-sample (no biological or technical replicate structure), since MaRR assesses reproducibility across replicate pairs

## Inputs

- Metabolite abundance matrix (features x replicate samples), e.g., RF-quantile pre-processed BioTech dataset R objects from the 'saved_objects' folder
- Raw data from Metabolomics Workbench (Project ID PR000438 for BioTech, PR000907 for Bio)
- MaRR functions 'source_MaRR.R' from the repository 'src' folder

## Outputs

- Marr() output object with MarrSamplepairs, MarrFeatures, MarrSamplepairsfiltered, and MarrFeaturesfiltered slots
- Histograms/distributions of percent reproducible features per sample pair and sample pairs per feature (via MarrPlotSamplepairs/MarrPlotFeatures)
- Scatter plots of MaRR reproducibility vs RSD for biological and technical replicates with 25% RSD cutoff annotation (reproducing Additional file 1: Figs. S24–S25)
- LaTeX comparison tables generated with xtable (reproducing Table S4), saved to the 'Output' folder

## How to apply

Load the replicate-level metabolite abundance matrix (features on rows, samples on columns), optionally the RF-quantile pre-processed object (imputation via BPCA/kNN plus quantile/median/RUV normalization) saved in the 'saved_objects' folder of the Ghoshlab/marr_paper_evaluations Real_Data repository. Source 'source_MaRR.R' from the 'src' folder (or use the `marr` Bioconductor package, requiring R >= 4.0) and run `Marr()` with the maximal rank statistic using 75% sample-pair reproducibility thresholds (pSamplepairs=0.75, pFeatures=0.75, alpha=0.05) to classify metabolites per sample pair and sample pairs per feature. Compute RSD of metabolite abundances across biological and technical replicates, then scatter-plot MaRR reproducibility versus RSD and overlay a 25% RSD cutoff to show whether it removes MaRR-identified irreproducible metabolites. Generate LaTeX summary tables with the `xtable` package, ordered as in MaRR_real_results.pdf, and write plots/tables to the 'Output' folder. The rationale is that MaRR makes no parametric assumptions on distributions or dependence structures, unlike CV/RSD, Spearman correlation, CAT plots, IDR, or correspondence curve regression.

## Related tools

- **marr** (R/Bioconductor package implementing the MaRR maximal rank statistic to compute reproducibility of features per sample pair and sample pairs per feature, with filtering thresholds) — http://bioconductor.org/packages/marr
- **marr_paper_evaluations** (Repository of scripts and saved R objects (Real_Data folder, Plot_R_Scripts, Table_R_Scripts, src/source_MaRR.R) to reproduce the MaRR vs RSD comparison plots and tables) — github.com/Ghoshlab/marr_paper_evaluations
- **xtable** (Generates the LaTeX code for the comparison tables in the same order as MaRR_real_results.pdf)
- **MSPrep** (GitHub-installed pre-processing package for the metabolomics pipelines preceding MaRR application) — KechrisLab/MSPrep

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.05)
```

## Evaluation signals

- Reproduced scatter plots of MaRR reproducibility vs RSD match the qualitative pattern of Additional file 1: Figs. S24–S25
- The xtable-generated Table S4 confirms that a 25% RSD cutoff does not remove any MaRR-identified irreproducible metabolites in the BioTech dataset
- Marr() runs with the paper's thresholds (pSamplepairs=0.75, pFeatures=0.75, alpha=0.05) return non-empty MarrSamplepairsfiltered and MarrFeaturesfiltered slots
- Reproducibility percentages align with published patterns: technical replicates show higher reproducibility than biological replicates (e.g., >70% reproducible metabolites per sample pair for technical layers)

## Limitations

- Real signals might fall in undetectable regions because of lack of separation/overlap between the reproducible and irreproducible components
- MaRR power drops for smaller values of the proportion of reproducible metabolites
- No comprehensive evaluation of pre-processing steps was performed; MaRR is intended as a benchmark for comparison, and data-dependent thresholds (c_s, c_m) should be chosen by inspecting reproducibility histograms

## Evidence

- [methods] marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [readme] Marr() thresholds and defaults: [evidence span withheld — anchor status: external_doc]
- [discussion] MaRR-based filtering rationale: "our MaRR based data-driven filtering approach has the ability to remove irreproducible metabolites and sample pairs while retaining reproducible metabolites and sample pairs based on simulations."
- [intro] existing reproducibility measures limited: "There are few approaches that specifically examine reproducibility for MS-based metabolomics experiments. The most common approach is calculating the coefficient of variation (CV) across replicates."
