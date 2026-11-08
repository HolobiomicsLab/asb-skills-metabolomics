---
name: replicate-reproducibility-estimation-mass-spectrometry
description: Use when MS-metabolomics data contain biological or technical replicate
  samples and you need a nonparametric, FDR-controlled measure of which metabolites
  and sample pairs are reproducible, or when comparing pre-processing pipelines.
when_to_use_negative:
- The goal is a comprehensive evaluation of pre-processing steps — the paper states
  the goal was not comprehensive evaluation of pre-processing, but to show the measure
  can be used as a benchmark for comparison
- The data have no biological or technical replicate structure, since MaRR assesses
  reproducibility across replicate pairs
- Separation between reproducible and irreproducible components is known to be poor
  — true signals may fall in undetectable regions due to overlap between components
tools:
- name: marr
  role: Implements the MaRR maximal rank statistic via the Marr() function, computing
    reproducibility of features per sample pair and sample pairs per feature, with
    filtering thresholds and plot functions
  repo: http://bioconductor.org/packages/marr
- name: MSPrep
  role: Pre-processes raw metabolomics data (missing value imputation with BPCA/kNN;
    normalization with quantile, median, RUV) before MaRR is applied
  repo: https://github.com/KechrisLab/MSPrep
- name: BiocManager
  role: Installs the stable marr release from Bioconductor under R >= 4.0
- name: devtools
  role: Installs the development versions of marr and MSPrep from GitHub
  repo: https://github.com/hadley/devtools
- name: xtable
  role: Generates LaTeX table code of reproducibility percentages in the same order
    as the tables in MaRR_real_results.pdf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:432c3359b1fe1e84a5e0e15f7adb9bcd4dac9b936a20a2acbb3cdc010df4e921
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# replicate-reproducibility-estimation-mass-spectrometry

## Summary

Applies the nonparametric Maximum Rank Reproducibility (MaRR) procedure, via the R/Bioconductor marr package, to estimate the percentage of reproducible metabolites per sample pair and reproducible sample pairs per metabolite in mass spectrometry metabolomics replicate experiments, including benchmarking of imputation-normalization pipelines.

## When to use

Use when MS-metabolomics data contain biological or technical replicate samples and you need a nonparametric, FDR-controlled measure of which metabolites and sample pairs are reproducible, or when comparing pre-processing pipelines.

## When NOT to use

- The goal is a comprehensive evaluation of pre-processing steps — the paper states the goal was not comprehensive evaluation of pre-processing, but to show the measure can be used as a benchmark for comparison
- The data have no biological or technical replicate structure, since MaRR assesses reproducibility across replicate pairs
- Separation between reproducible and irreproducible components is known to be poor — true signals may fall in undetectable regions due to overlap between components

## Inputs

- Processed metabolomics feature matrix (metabolites as rows, replicate samples as columns); data frame, matrix, or SummarizedExperiment with one assay
- Raw Metabolomics Workbench datasets (e.g., BioTech dataset, Project ID PR000438; Bio dataset, Project ID PR000907) prior to MSPrep pre-processing
- Optional threshold parameters: pSamplepairs and pFeatures in (0,1) and FDR level alpha

## Outputs

- Distribution of percent reproducible metabolites (features) per sample pair (MarrSamplepairs)
- Distribution of percent reproducible sample pairs per metabolite (MarrFeatures)
- Filtered percentages of reproducible features and sample pairs at chosen thresholds (MarrSamplepairsfiltered, MarrFeaturesfiltered)
- Histogram plots of reproducibility distributions; LaTeX tables (via xtable) comparing pipelines

## How to apply

Install marr from Bioconductor (BiocManager::install('marr'), requires R >= 4.0) and MSPrep from GitHub (KechrisLab/MSPrep, ref='feature-generalize') for pre-processing. Preprocess the replicate matrix under the chosen pipeline(s) — missing-value imputation with BPCA or kNN crossed with quantile, median, or RUV normalization (6 combinations for the BioTech benchmark). Arrange data as metabolites on rows and samples on columns, then run Marr() with thresholds pSamplepairs and pFeatures (default 0.75 each) and FDR level alpha (default 0.05) to obtain the distributions of percent reproducible features per sample pair and sample pairs per feature. Use the histogram plot functions (MarrPlotFeatures, MarrPlotSamplepairs) to select data-dependent thresholds (c_s, c_m) for identifying reproducible metabolites and sample pairs. To compare pipelines (e.g., for the BioTech dataset, Project ID PR000438), aggregate the reproducibility percentages across all 6 imputation-normalization combinations and tabulate them (e.g., LaTeX via xtable); the maximal reproducibility combination (RF-median or RF-quantile) is the best-performing pipeline.

## Related tools

- **marr** (Implements the MaRR maximal rank statistic via the Marr() function, computing reproducibility of features per sample pair and sample pairs per feature, with filtering thresholds and plot functions) — http://bioconductor.org/packages/marr
- **MSPrep** (Pre-processes raw metabolomics data (missing value imputation with BPCA/kNN; normalization with quantile, median, RUV) before MaRR is applied) — https://github.com/KechrisLab/MSPrep
- **BiocManager** (Installs the stable marr release from Bioconductor under R >= 4.0)
- **devtools** (Installs the development versions of marr and MSPrep from GitHub) — https://github.com/hadley/devtools
- **xtable** (Generates LaTeX table code of reproducibility percentages in the same order as the tables in MaRR_real_results.pdf)

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.05)
```

## Evaluation signals

- For the BioTech dataset, the percentage of reproducible metabolites per sample pair for technical replicates must exceed 70% and be much higher than for biological replicates
- The tabulated reproducibility percentages must match the paper's reported BioTech results in Table 2, where RF-median or RF-quantile achieve maximum reproducibility, outperforming BPCA- and kNN-based combinations
- Threshold parameters pSamplepairs and pFeatures must lie between 0 and 1; alpha controls FDR (default 0.05)
- The MaRR procedure should control the False Discovery Rate under gradual reduction in correlation between replicate pairs for less highly ranked signals
- Bias (estimated minus true reproducible signal proportion) should be close to zero when validated on simulations (SIM_B1.R ... SIM_D4.R objects)

## Limitations

- No comprehensive evaluation of pre-processing steps was performed; the goal was only to benchmark comparisons of imputation-normalization pipelines
- True signals may fall in undetectable regions due to overlap between reproducible and irreproducible components
- MaRR tends to have lower power for detecting reproducible metabolites when the proportion of reproducible metabolites is small
- Marr() requires the data object with observations (metabolites/genes) on rows and samples on columns; installation requires R >= 4.0

## Evidence

- [methods] marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [readme] Marr() function signature and defaults: [evidence span withheld — anchor status: external_doc]
- [discussion] BioTech validation threshold: "For the BioTech data set (Fig. 6), the percentage of reproducible metabolites per sample pair greater than 70% for technical replicates is much higher compared to their biological replicates"
- [discussion] Pipeline comparison scope: "our goal was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison."
- [methods] Imputation-normalization combinations: "Imputation- BPCA and kNN and Normalization- quantile, median and RUV"
- [methods] MSPrep installation: "install_github("KechrisLab/MSPrep",ref="feature-generalize")"
