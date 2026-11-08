---
name: fdr-threshold-based-reproducible-feature-selection
description: Use when you have MS-Metabolomics replicate data (technical or biological
  replicates, e.g. COPDGene Tech/BioTech/Bio) and need to flag which metabolites and
  sample pairs are reproducible while controlling FDR.
when_to_use_negative:
- When the goal is a comprehensive evaluation of pre-processing steps (BPCA/kNN imputation,
  quantile/median/RUV normalization) — the article states the goal was not comprehensive
  evaluation but only benchmarking comparisons
- When the data contain a very small proportion of reproducible metabolites — MaRR
  tends to have lower power in that regime
- When reproducible and irreproducible components strongly overlap — true signals
  may fall in undetectable regions due to lack of separation between components
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: marr
  role: Implements the MaRR maximal rank statistic, FDR control, reproducibility-score
    filtering, and histogram plotting of features per sample pair and sample pairs
    per feature
  repo: http://bioconductor.org/packages/marr
- name: R
  role: Runtime environment (>=4.0) in which marr and the MaRR helper scripts are
    executed
- name: BiocManager
  role: Installer used to obtain the stable marr release from Bioconductor
- name: MSPrep
  role: Pre-processing (imputation/normalization) of MS-Metabolomics data prior to
    MaRR-based filtering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:78cd8ac2eb60ed8de73e2d692cdf7105d7a3bdb4d8939f8bb51ddb7d1128cbf2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - MSPrep
  - R
  - marr
  - xtable
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# fdr-threshold-based-reproducible-feature-selection

## Summary

Identifies reproducible metabolites and sample pairs in replicate MS-Metabolomics experiments using the nonparametric MaRR maximal rank statistic (marr R/Bioconductor package), declaring features reproducible at a conservative FDR level with data-dependent thresholds on reproducibility score distributions.

## When to use

Use when you have MS-Metabolomics replicate data (technical or biological replicates, e.g. COPDGene Tech/BioTech/Bio) and need to flag which metabolites and sample pairs are reproducible while controlling FDR.

## When NOT to use

- When the goal is a comprehensive evaluation of pre-processing steps (BPCA/kNN imputation, quantile/median/RUV normalization) — the article states the goal was not comprehensive evaluation but only benchmarking comparisons
- When the data contain a very small proportion of reproducible metabolites — MaRR tends to have lower power in that regime
- When reproducible and irreproducible components strongly overlap — true signals may fall in undetectable regions due to lack of separation between components

## Inputs

- Features-by-samples matrix or data frame (metabolites as rows, samples as columns) from replicate MS-Metabolomics data, optionally a SummarizedExperiment with one assay
- Pre-processed MS-Metabolomics data (e.g. after imputation with BPCA/kNN and normalization with quantile/median/RUV)
- FDR level alpha (e.g. 0.01) and threshold values pSamplepairs / pFeatures (defaults 0.75) or histogram-guided c_s and c_m thresholds

## Outputs

- Distribution of percent reproducible features (metabolites) per sample pair (MarrSamplepairs)
- Distribution of percent reproducible sample pairs per feature (MarrFeatures)
- Filtered percent of reproducible features and sample pairs at the chosen threshold (MarrSamplepairsfiltered, MarrFeaturesfiltered)
- Histograms of reproducibility scores via MarrPlotSamplepairs() and MarrPlotFeatures()

## How to apply

Run the marr maximal rank statistic on a features-by-samples matrix (or SummarizedExperiment) to estimate reproducibility of metabolites per sample pair and sample pairs per metabolite. Declare signals reproducible at a conservative FDR level (the real-data analysis used alpha=0.01), with default thresholds pSamplepairs=0.75 and pFeatures=0.75 in Marr(). Select data-dependent threshold values c_s and c_m by visualizing histograms of the marr reproducibility scores, since threshold choice depends on the shape of each dataset's score distributions. Extract filtered results with MarrSamplepairsfiltered() and MarrFeaturesfiltered(), then summarize the percentages of reproducible metabolites per sample pair and sample pairs per metabolite above cutoffs (e.g. >70%, >80%, >90%) per replicate layer. The rationale is that MaRR is nonparametric — it makes no assumptions on the distributions or dependence structures of reproducible metabolites — and controls FDR when correlation between replicate pairs gradually decreases for less highly ranked signals.

## Related tools

- **marr** (Implements the MaRR maximal rank statistic, FDR control, reproducibility-score filtering, and histogram plotting of features per sample pair and sample pairs per feature) — http://bioconductor.org/packages/marr
- **R** (Runtime environment (>=4.0) in which marr and the MaRR helper scripts are executed)
- **BiocManager** (Installer used to obtain the stable marr release from Bioconductor)
- **MSPrep** (Pre-processing (imputation/normalization) of MS-Metabolomics data prior to MaRR-based filtering)

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs = 0.75, pFeatures = 0.75, alpha = 0.01); MarrSamplepairsfiltered(MarrOutput)
```

## Evaluation signals

- Reproduced percentages match Table 1, e.g. BioTech technical replicates at >70% reproducible metabolites per sample pair give 90.08% vs 49.85% for biological replicates
- FDR is controlled at the declared level (real data used the conservative alpha=0.01)
- Filtered percentages are consistent with the extracted MarrSamplepairsfiltered/MarrFeaturesfiltered values for the chosen pSamplepairs/pFeatures thresholds
- Chosen thresholds c_s and c_m are justified by visual inspection of histograms of the marr reproducibility scores
- Bias between estimated and true reproducible-signal proportions is close to zero in the accompanying simulations

## Limitations

- True signals may fall in undetectable regions because of overlapping between the reproducible and irreproducible components
- MaRR tends to have low power when the proportion of reproducible metabolites is small
- The approach was used to benchmark pre-processing pipelines, not to perform a comprehensive evaluation of pre-processing steps

## Evidence

- [methods] marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [discussion] data-driven filtering retains reproducible metabolites: "our MaRR based data-driven filtering approach has the ability to remove irreproducible metabolites and sample pairs while retaining reproducible metabolites and sample pairs based on simulations."
- [discussion] histogram-guided threshold selection: [evidence span withheld — anchor status: not_found]
- [readme] Marr() optional thresholds and FDR level: "alpha (Optional)
level of significance to control the False Discovery Rate (FDR). Default
is 0.05."
- [readme] accepted input object types for Marr(): "object: a data frame or a matrix or a Summarized Experiment with one
assay object with observations (e.g., metabolites or genes) on the rows"
