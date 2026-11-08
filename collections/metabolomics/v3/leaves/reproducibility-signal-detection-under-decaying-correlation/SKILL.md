---
name: reproducibility-signal-detection-under-decaying-correlation
description: Use when assessing reproducibility of high-dimensional replicate experiments
  (e.g., MS-metabolomics feature tables with metabolites x replicate sample pairs)
  where replicate correlation decays with rank, and FDR-controlled, power-assessed
  detection of reproducible signals is needed.
when_to_use_negative:
- When a comprehensive evaluation of pre-processing steps (imputation/normalization)
  is the goal — the article states the goal was only to benchmark comparisons, not
  comprehensive pre-processing evaluation
- When reproducible and irreproducible components lack separation — true signals may
  fall in undetectable regions due to overlap between components
- When the input is not replicate data amenable to pairwise rank-based comparison
  (e.g., a single-sample feature table with no replicates)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: marr
  role: Nonparametric Maximum Rank Reproducibility method implementing the maximal
    rank statistic to detect reproducible signals; Marr() function with pSamplepairs,
    pFeatures, alpha parameters
  repo: http://bioconductor.org/packages/marr
- name: R
  role: Execution environment (R >= 4.0 required) for running simulation scripts,
    MaRR functions, and plotting scripts
- name: devtools
  role: Used to install the development version of marr from GitHub (devtools::install_github("Ghoshlab/marr"))
  repo: https://github.com/Ghoshlab/marr
- name: BiocManager
  role: Installer for the stable Bioconductor release of the marr package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:229f7edbe9cebaf74ec69706dfecd30d177ecefe61cb0ffc23c86202e9c93621
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - R
  - devtools
  - marr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# reproducibility-signal-detection-under-decaying-correlation

## Summary

Applies the nonparametric MaRR (Maximum Rank Reproducibility) maximal rank statistic to estimate the proportion of reproducible signals and detect reproducible features when correlation between replicate pairs decreases gradually with rank, without parametric assumptions on distributions or dependence structures.

## When to use

Use when assessing reproducibility of high-dimensional replicate experiments (e.g., MS-metabolomics feature tables with metabolites x replicate sample pairs) where replicate correlation decays with rank, and FDR-controlled, power-assessed detection of reproducible signals is needed.

## When NOT to use

- When a comprehensive evaluation of pre-processing steps (imputation/normalization) is the goal — the article states the goal was only to benchmark comparisons, not comprehensive pre-processing evaluation
- When reproducible and irreproducible components lack separation — true signals may fall in undetectable regions due to overlap between components
- When the input is not replicate data amenable to pairwise rank-based comparison (e.g., a single-sample feature table with no replicates)

## Inputs

- High-dimensional data matrix or SummarizedExperiment with one assay: observations (metabolites/genes) on rows, replicate samples on columns
- Simulated replicate-pair datasets with correlation decreasing linearly with rank (Simulation II settings: pi_1 in {0.2, 0.4, 0.75, 0.9}, r_0 in {0.4, 0.6, 0.99})
- FDR significance level alpha (default 0.05)
- Optional reproducibility thresholds pSamplepairs and pFeatures (default 0.75)

## Outputs

- Estimated proportion of reproducible signals per dataset
- Per-feature distribution of percent reproducible sample pairs and per-sample-pair distribution of percent reproducible features
- Aggregated empirical FDR, power (1-NDR), and bias per simulation setting (R objects like 'B1_sim_results.Rdata')
- FDR, power, and bias plots (Figs. 3-4, S6-S7 equivalents) via SIMPLOT scripts
- Lists of reproducible metabolites and sample pairs after thresholding

## How to apply

Source the MaRR functions ('source_MaRR.R' from the repository 'src' folder) in R (>=4.0), or install the Bioconductor 'marr' package via BiocManager. Generate simulated datasets under the target correlation regime (e.g., Simulation II: pi_1 in {0.2, 0.4, 0.75, 0.9} crossed with r_0 in {0.4, 0.6, 0.99}, with replicate correlation decreasing linearly with rank, 1000 replicates per setting) using the SIM_B1.R through SIM_D4.R scripts. Apply the nonparametric MaRR maximal rank statistic to each dataset to estimate the proportion of reproducible signals and detect reproducible features per sample pair, controlling FDR at the nominal alpha level (default 0.05 in the Marr() function). Aggregate empirical FDR, power (1-NDR), and bias (estimated minus true reproducible proportion) per setting, save as R objects (e.g., 'B1_sim_results.Rdata' ... 'D4_sim_results.Rdata'), and plot with the design-specific script (e.g., 'SIMPLOT_B.R'). For real data, apply Marr() to a matrix/SummarizedExperiment with metabolites on rows and replicate sample pairs on columns, optionally selecting data-dependent thresholds pSamplepairs and pFeatures (default 0.75) guided by the reproducibility histograms.

## Related tools

- **marr** (Nonparametric Maximum Rank Reproducibility method implementing the maximal rank statistic to detect reproducible signals; Marr() function with pSamplepairs, pFeatures, alpha parameters) — http://bioconductor.org/packages/marr
- **R** (Execution environment (R >= 4.0 required) for running simulation scripts, MaRR functions, and plotting scripts)
- **devtools** (Used to install the development version of marr from GitHub (devtools::install_github("Ghoshlab/marr"))) — https://github.com/Ghoshlab/marr
- **BiocManager** (Installer for the stable Bioconductor release of the marr package)

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs = 0.75, pFeatures = 0.75, alpha = 0.05)
```

## Evaluation signals

- Reproduced FDR plots show empirical FDR at or below the nominal level across all 12 Simulation II settings, with conservative behavior when pi_1 is high and r_0 is low
- Power (1-NDR) is high across settings except at the smallest pi_1 values (smallest proportions of reproducible metabolites)
- Bias (estimated minus true reproducible signal proportion) is close to zero, with the expected negative-bias pattern at low pi_1
- Figures match Figs. 3-4 and S6-S7 of the paper when SIMPLOT_B.R is run on the saved simulation R objects
- MarrSamplepairs() and MarrFeatures() accessors return distributions, and MarrSamplepairsfiltered()/MarrFeaturesfiltered() return percents consistent with the chosen thresholds

## Limitations

- MaRR tends to have lower power for detecting reproducible metabolites when the true proportion of reproducible signals (pi_1) is small
- Real signals might fall in undetectable regions because of overlapping between the reproducible and irreproducible components (lack of separation)
- FDR control can be conservative when pi_1 is high and r_0 is low
- No comprehensive evaluation of pre-processing steps was performed; the method was only benchmarked for comparison of pipelines (imputation: BPCA, kNN; normalization: quantile, median, RUV)

## Evidence

- [methods] marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [methods] The scripts SIM_B1.R..SIM_D4.R generate corresponding R objects: "The scripts 'SIM_B1.R',.. , 'SIM_C1.R',..., 'SIM_D4.R' can generate the corresponding R objects 'B1_sim_results.Rdata',..,'C1_sim_results.Rdata',...,'D4_sim_results.Rdata'"
- [abstract] The MaRR procedure effectively controls the FDR under gradual correlation reduction: "the MaRR procedure effectively controls the False Discovery Rate (FDR) when there was a gradual reduction in correlation between replicate pairs for less highly ranked signals"
- [abstract] High power except for smaller proportions of reproducible metabolites: "the MaRR procedure tends to have high power for detecting reproducible metabolites in most situations except for smaller values of proportion of reproducible metabolites"
- [readme] README: main function and thresholds: [evidence span withheld — anchor status: external_doc]
- [discussion] Undetectable regions due to overlapping components: "the real signals might fall in the undetectable regions because of overlapping between the reproducible and irreproducible components"
