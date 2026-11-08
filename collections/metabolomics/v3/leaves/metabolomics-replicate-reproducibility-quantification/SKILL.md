---
name: metabolomics-replicate-reproducibility-quantification
description: Use when high-dimensional metabolomics (or similar omics) data contain
  technical or biological replicate samples and you need to identify reproducible
  metabolites/sample pairs and compare replicate layers, e.g. Tech vs BioTech vs Bio
  COPDGene datasets.
when_to_use_negative:
- The goal is a comprehensive evaluation of pre-processing steps — the authors state
  'our goal was not to do a comprehensive evaluation of pre-processing steps, but
  instead to show how this measure can be used as a benchmark for comparison'
- Data contain no replicate structure (single measurement per condition), since MaRR
  assesses reproducibility across replicate sample pairs
- You need parametric modeling of the dependence structure of reproducible signals
  — MaRR's advantage is precisely that it avoids parametric assumptions, not that
  it models them
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: marr
  role: Implements the MaRR maximal rank statistic via Marr() and plotting/accessor
    functions to quantify reproducibility of features per sample pair and sample pairs
    per feature
  repo: http://bioconductor.org/packages/marr
- name: BiocManager
  role: Installs the stable marr release from Bioconductor (requires R >= 4.0)
- name: MSPrep
  role: Pre-processing of MS-Metabolomics data (feature-level generalization branch)
    prior to applying MaRR
  repo: https://github.com/KechrisLab/MSPrep
- name: xtable
  role: Generates the LaTeX code for reproducibility summary tables (Table 1)
- name: devtools
  role: Installs the development version of marr and MSPrep from GitHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:56c730345db66476a71771cfa9dd6ba55134d2f9bb717dbc7c7b8c8d68e052ad
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# metabolomics-replicate-reproducibility-quantification

## Summary

Quantify reproducibility of metabolites per sample pair and sample pairs per metabolite in MS-Metabolomics replicate experiments using the nonparametric MaRR (Maximum Rank Reproducibility) maximal rank statistic, implemented in the R/Bioconductor marr package.

## When to use

Use when high-dimensional metabolomics (or similar omics) data contain technical or biological replicate samples and you need to identify reproducible metabolites/sample pairs and compare replicate layers, e.g. Tech vs BioTech vs Bio COPDGene datasets.

## When NOT to use

- The goal is a comprehensive evaluation of pre-processing steps — the authors state 'our goal was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison'
- Data contain no replicate structure (single measurement per condition), since MaRR assesses reproducibility across replicate sample pairs
- You need parametric modeling of the dependence structure of reproducible signals — MaRR's advantage is precisely that it avoids parametric assumptions, not that it models them

## Inputs

- Preprocessed metabolite-by-sample matrix (metabolites on rows, samples on columns) as a data frame, matrix, or SummarizedExperiment with one assay
- Tech, BioTech, and Bio COPDGene MS-Metabolomics data objects (Real_Data 'data' folder; benchmarking files from Metabolomics Workbench, Project IDs PR000438 and PR000907)
- Choice of pre-processing pipeline: BPCA or kNN imputation plus quantile, median, or RUV normalization

## Outputs

- Distribution of percent reproducible sample pairs per feature (MarrFeatures) and percent reproducible features per sample pair (MarrSamplepairs)
- Percentages of reproducible metabolites/sample pairs above >70%, >80%, and >90% thresholds (MarrSamplepairsfiltered, MarrFeaturesfiltered)
- Histograms of reproducibility scores (MarrPlotFeatures, MarrPlotSamplepairs) used to pick c_s and c_m thresholds
- LaTeX summary tables generated with xtable (assembled in MaRR_real_results.pdf); intermediate *.Rdata objects in 'saved_objects'

## How to apply

Load preprocessed metabolite-by-sample matrices (rows = metabolites, columns = samples) after one of the 6 pre-processing pipelines (BPCA or kNN imputation combined with quantile, median, or RUV normalization). Run the Marr() function with a data frame, matrix, or SummarizedExperiment (one assay), setting the FDR level alpha=0.01 (conservative for real MS-Metabolomics data) and data-dependent thresholds pSamplepairs/pFeatures (c_s and c_m, default 0.75) chosen by inspecting histograms of the reproducibility scores via MarrPlotFeatures()/MarrPlotSamplepairs(). Extract MarrSamplepairs, MarrFeatures, MarrSamplepairsfiltered, and MarrFeaturesfiltered to compute the percentage of reproducible metabolites per sample pair and reproducible sample pairs per metabolite above >70%, >80%, and >90% cutoffs for technical vs biological replicate layers. Compare these percentages across replicate layers (e.g., technical replicates show much higher reproducibility than biological replicates) and render summary tables via xtable. MaRR is preferred over CV/RSD, Spearman correlation, CAT plots, IDR, or correspondence curve regression because it makes no parametric assumptions on the distributions or dependence structures of reproducible metabolites and controls FDR under gradual correlation decay.

## Related tools

- **marr** (Implements the MaRR maximal rank statistic via Marr() and plotting/accessor functions to quantify reproducibility of features per sample pair and sample pairs per feature) — http://bioconductor.org/packages/marr
- **BiocManager** (Installs the stable marr release from Bioconductor (requires R >= 4.0))
- **MSPrep** (Pre-processing of MS-Metabolomics data (feature-level generalization branch) prior to applying MaRR) — https://github.com/KechrisLab/MSPrep
- **xtable** (Generates the LaTeX code for reproducibility summary tables (Table 1))
- **devtools** (Installs the development version of marr and MSPrep from GitHub)

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.05)
```

## Evaluation signals

- Reproducing the BioTech dataset should yield 90.08% of metabolites per sample pair with >70% reproducibility for technical replicates vs 49.85% for biological replicates, matching Table 1
- FDR is controlled at the declared level alpha=0.01 for the real-data analysis; simulation bias (estimated minus true reproducible-signal proportion) should be close to zero
- The 18 R scripts (6 pre-processing pipelines x 3 datasets) each produce a saved R object in 'saved_objects' (*.Rdata)
- Thresholds c_s and c_m are data-dependent: histogram visualizations of marr reproducibility scores must be inspected before fixing pSamplepairs/pFeatures
- Technical-replicate reproducibility should exceed biological-replicate reproducibility across all three datasets (Tech, BioTech, Bio)

## Limitations

- MaRR tends to have high power except for smaller values of the proportion of reproducible metabolites
- True signals may fall in undetectable regions due to overlap/ lack of separation between the reproducible and irreproducible components
- The 6 pre-processing pipelines were benchmarked only, not comprehensively evaluated
- Real-data signals might overlap with irreproducible components, limiting detection in some scenarios

## Evidence

- [methods] Nonparametric MaRR detects reproducible signals: "marr (Maximum Rank Reproducibility) is a nonparametric approach that detects reproducible signals using a maximal rank statistic for high-dimensional biological data"
- [abstract] Higher technical vs biological reproducibility: "a higher level of reproducibility for technical replicates compared to biological replicates across all the three different datasets"
- [discussion] Data-dependent thresholds via histograms: [evidence span withheld — anchor status: not_found]
- [readme] Marr() input format and defaults: [evidence span withheld — anchor status: external_doc]
- [abstract] Bioconductor availability: "the open-source R package marr, which is freely available from Bioconductor"
- [readme] Plotting functions for reproducibility scores: [evidence span withheld — anchor status: external_doc]
