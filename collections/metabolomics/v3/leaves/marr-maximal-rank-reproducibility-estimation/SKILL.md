---
name: marr-maximal-rank-reproducibility-estimation
description: Use when assessing reproducibility of metabolites (or other features)
  across replicate experiments — e.g., comparing technical vs biological replicate
  reproducibility in MS-Metabolomics data — without parametric assumptions.
when_to_use_negative:
- When the goal is a comprehensive evaluation of pre-processing steps rather than
  a benchmark comparison — the authors state 'our goal was not to do a comprehensive
  evaluation of pre-processing steps, but instead to show how this measure can be
  used as a benchmark for comparison.'
- When the proportion of reproducible metabolites is expected to be small — simulations
  showed MaRR tends to have high power 'except for smaller values of proportion of
  reproducible metabolites'.
- When data has strong overlap between reproducible and irreproducible components
  — true signals may fall in undetectable regions due to lack of separation.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: marr
  role: R/Bioconductor package implementing the MaRR maximal rank statistic; Marr()
    computes reproducibility of features per sample pair and sample pairs per feature
  repo: http://bioconductor.org/packages/marr
- name: BiocManager
  role: Installs the stable marr release from Bioconductor
- name: devtools
  role: Installs the development version of marr (Ghoshlab/marr) from GitHub
  repo: https://github.com/Ghoshlab/marr
- name: MSPrep
  role: Pre-processing (imputation/normalization) of MS-Metabolomics data before applying
    MaRR; installed via install_github('KechrisLab/MSPrep', ref='feature-generalize')
  repo: https://github.com/KechrisLab/MSPrep
- name: xtable
  role: Generates LaTeX code for reproducibility summary tables in the evaluation
    scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:a0e8a547db952f6f2b31f2d51e56f01f9c2d87409440706de119c5ad5dce0468
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
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 22
---

# marr-maximal-rank-reproducibility-estimation

## Summary

Applies the nonparametric MaRR (Maximum Rank Reproducibility) procedure via the marr R/Bioconductor package to estimate reproducibility of features per sample pair and sample pairs per feature in high-dimensional biological replicate experiments, such as COPDGene MS-Metabolomics technical and biological replicate data.

## When to use

Use when assessing reproducibility of metabolites (or other features) across replicate experiments — e.g., comparing technical vs biological replicate reproducibility in MS-Metabolomics data — without parametric assumptions.

## When NOT to use

- When the goal is a comprehensive evaluation of pre-processing steps rather than a benchmark comparison — the authors state 'our goal was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison.'
- When the proportion of reproducible metabolites is expected to be small — simulations showed MaRR tends to have high power 'except for smaller values of proportion of reproducible metabolites'.
- When data has strong overlap between reproducible and irreproducible components — true signals may fall in undetectable regions due to lack of separation.

## Inputs

- Data frame, matrix, or SummarizedExperiment (one assay) with observations (metabolites/genes) on rows and samples on columns (e.g., Tech/BioTech/Bio COPDGene MS-Metabolomics objects from the Real_Data 'data' folder)
- Threshold values pSamplepairs and pFeatures (0–1, default 0.75) and FDR level alpha (e.g., 0.01 for real data)

## Outputs

- Distribution of percent reproducible features per sample pair (MarrSamplepairs)
- Distribution of percent reproducible sample pairs per feature (MarrFeatures)
- Percent of reproducible features/sample pairs passing the threshold (MarrSamplepairsfiltered, MarrFeaturesfiltered)
- Histogram plots of reproducibility scores via MarrPlotSamplepairs()/MarrPlotFeatures()
- LaTeX tables (via xtable) of percentages above >70%, >80%, >90% cutoffs, assembled into MaRR_real_results.pdf

## How to apply

Install the marr package from Bioconductor (BiocManager::install('marr'), requiring R >= 4.0) or the development version from GitHub, then call Marr() on a data frame/matrix/SummarizedExperiment with metabolites on rows and samples on columns. Set the FDR level alpha (default 0.05; the paper used a conservative alpha=0.01 for real MS-Metabolomics data) and threshold values pSamplepairs and pFeatures (default 0.75), choosing data-dependent thresholds c_s and c_m guided by histogram visualizations of the marr reproducibility scores. The maximal rank statistic yields distributions of percent reproducible features per sample pair (MarrSamplepairs) and percent reproducible sample pairs per feature (MarrFeatures), plus filtered percentages at the chosen thresholds. When benchmarking pre-processing pipelines, apply MaRR after each of 6 pipelines (BPCA/kNN imputation combined with quantile, median, or RUV normalization) so reproducibility comparisons are like-for-like. Summarize the percent of reproducible metabolites per sample pair above cutoffs such as >70%, >80%, and >90% for each replicate layer (e.g., BioTech: 90.08% technical vs 49.85% biological at >70%).

## Related tools

- **marr** (R/Bioconductor package implementing the MaRR maximal rank statistic; Marr() computes reproducibility of features per sample pair and sample pairs per feature) — http://bioconductor.org/packages/marr
- **BiocManager** (Installs the stable marr release from Bioconductor)
- **devtools** (Installs the development version of marr (Ghoshlab/marr) from GitHub) — https://github.com/Ghoshlab/marr
- **MSPrep** (Pre-processing (imputation/normalization) of MS-Metabolomics data before applying MaRR; installed via install_github('KechrisLab/MSPrep', ref='feature-generalize')) — https://github.com/KechrisLab/MSPrep
- **xtable** (Generates LaTeX code for reproducibility summary tables in the evaluation scripts)

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.05)
```

## Evaluation signals

- Reproduced Table 1 percentages match published values, e.g., BioTech dataset: 90.08% of technical replicate metabolites per sample pair reproducible at >70% vs 49.85% for biological replicates
- FDR is controlled at the declared alpha (0.01 for real data); simulation bias (estimated minus true reproducible signal proportion) is close to zero
- The 18 R scripts (6 pre-processing pipelines x 3 datasets) each produce a corresponding saved R object in 'saved_objects' (*.Rdata)
- Histogram visualizations of marr reproducibility scores support the chosen data-dependent thresholds c_s and c_m
- Technical replicate reproducibility exceeds biological replicate reproducibility across all three COPDGene datasets (Tech, BioTech, Bio)

## Limitations

- True signals might fall in undetectable regions because of overlapping between the reproducible and irreproducible components (lack of separation).
- MaRR tends to have high power except for smaller values of the proportion of reproducible metabolites.
- The MaRR evaluation was not a comprehensive assessment of pre-processing steps (BPCA/kNN imputation; quantile/median/RUV normalization) — it only benchmarks comparisons.
- Real data results adopted a conservative approach (alpha=0.01) to identify reproducible signals.

## Evidence

- [methods] marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [readme] Marr() function inputs and thresholds: [evidence span withheld — anchor status: external_doc]
- [discussion] Real data conservative FDR and data-dependent thresholds: "For MS-Metabolomics real data, we adopted a more conservative approach to identify reproducible signals"
- [discussion] Data-dependent threshold selection with histograms: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmarking result confirming technical vs biological reproducibility: "For the BioTech data set (Fig. 6), the percentage of reproducible metabolites per sample pair greater than 70% for technical replicates is much higher compared to their biological replicates"
- [discussion] Pre-processing pipeline scope: "our goal was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison."
