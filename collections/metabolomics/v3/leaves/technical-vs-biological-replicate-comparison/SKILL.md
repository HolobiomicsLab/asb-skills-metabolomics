---
name: technical-vs-biological-replicate-comparison
description: Use when comparing reproducibility between technical and biological replicate
  layers of MS-Metabolomics feature matrices (e.g., COPDGene Tech, BioTech PR000438,
  Bio PR000907) to quantify reproducible metabolites per sample pair and sample pairs
  per metabolite.
when_to_use_negative:
- The goal is a comprehensive evaluation of pre-processing pipelines — the authors
  state 'our goal was not to do a comprehensive evaluation of pre-processing steps,
  but instead to show how this measure can be used as a benchmark for comparison'
- The data contain no technical or biological replicate sample pairs, so the maximal
  rank statistic across replicate pairs cannot be estimated
- Input is raw mass-spectrometry data; MaRR is applied only after data processing
  (imputation and normalization) to the metabolite-by-sample matrix
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: marr
  role: Implements the Maximum Rank Reproducibility maximal rank statistic; the Marr()
    function computes reproducibility of features per sample pair and sample pairs
    per feature with thresholds pSamplepairs, pFeatures, and FDR level alpha
  repo: http://bioconductor.org/packages/marr
- name: BiocManager
  role: Installs the stable Bioconductor release of the marr package
- name: MSPrep
  role: Feature-generalize branch installed from GitHub for MS-Metabolomics data pre-processing
    in the evaluation scripts
  repo: https://github.com/KechrisLab/MSPrep
- name: xtable
  role: Generates the LaTeX code for the reproducibility summary tables
- name: R (>=4.0)
  role: Runtime environment for all marr/MSPrep evaluation scripts (requires R version
    4.0 or later)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:d4b32bcdacdf8c244d28879a013bb01027d96abf0b8451a60bc43f34d51f8647
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - MSPrep
  - R
  - R (>=4.0)
  - marr
  - xtable
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5517241379310345
n_claims: 29
---

# technical-vs-biological-replicate-comparison

## Summary

Applies the nonparametric Maximum Rank Reproducibility (MaRR) procedure to estimate, per sample pair and per metabolite, whether technical replicate layers of MS-Metabolomics data (e.g., COPDGene Tech, BioTech, Bio) show higher reproducibility than biological replicate layers.

## When to use

Use when comparing reproducibility between technical and biological replicate layers of MS-Metabolomics feature matrices (e.g., COPDGene Tech, BioTech PR000438, Bio PR000907) to quantify reproducible metabolites per sample pair and sample pairs per metabolite.

## When NOT to use

- The goal is a comprehensive evaluation of pre-processing pipelines — the authors state 'our goal was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison'
- The data contain no technical or biological replicate sample pairs, so the maximal rank statistic across replicate pairs cannot be estimated
- Input is raw mass-spectrometry data; MaRR is applied only after data processing (imputation and normalization) to the metabolite-by-sample matrix

## Inputs

- Pre-processed MS-Metabolomics feature matrices (metabolites as rows, samples as columns) from the Tech, BioTech (Project PR000438), and Bio (Project PR000907) COPDGene datasets
- Data objects from the marr_paper_evaluations repository's Real_Data 'data' folder
- Pre-processing pipeline settings: BPCA or kNN imputation combined with quantile, median, or RUV normalization

## Outputs

- marr reproducibility score distributions for features per sample pair (MarrSamplepairs) and sample pairs per feature (MarrFeatures), with filtered percentages (MarrSamplepairsfiltered, MarrFeaturesfiltered)
- Saved R objects (*.Rdata) in the 'saved_objects' folder
- Percentages of reproducible metabolites per sample pair and sample pairs per metabolite above >70%, >80%, and >90% cutoffs per replicate layer, tabulated as LaTeX via xtable
- Plots and summary tables assembled in the PDF 'MaRR_real_results.pdf'

## How to apply

Load the Tech/BioTech/Bio data objects from the repository's Real_Data 'data' folder (benchmarking data also downloadable from Metabolomics Workbench, Project IDs PR000438 and PR000907), and install the marr package from Bioconductor via BiocManager::install("marr") (requires R >= 4.0); source the MaRR helper functions from 'src/source_MaRR.R'. For each dataset, run the maximal rank statistic across the 6 pre-processing pipelines (BPCA/kNN imputation combined with quantile/median/RUV normalization) using the 18 R_Scripts, saving intermediates as *.Rdata in 'saved_objects'. Declare metabolites and sample pairs reproducible at the conservative FDR level alpha = 0.01, choosing data-dependent thresholds c_s (sample pairs per metabolite) and c_m (metabolites per sample pair) guided by histograms of the marr reproducibility scores rather than the default 0.75. Summarize the percentages of reproducible metabolites per sample pair above the >70%, >80%, and >90% cutoffs for the technical and biological replicate layers, generating LaTeX tables with xtable (assembled in MaRR_real_results.pdf). The rationale is that MaRR makes no parametric assumptions on the distributions or dependence structures of reproducible metabolites, and the expected pattern is higher technical than biological reproducibility across all three datasets.

## Related tools

- **marr** (Implements the Maximum Rank Reproducibility maximal rank statistic; the Marr() function computes reproducibility of features per sample pair and sample pairs per feature with thresholds pSamplepairs, pFeatures, and FDR level alpha) — http://bioconductor.org/packages/marr
- **BiocManager** (Installs the stable Bioconductor release of the marr package)
- **MSPrep** (Feature-generalize branch installed from GitHub for MS-Metabolomics data pre-processing in the evaluation scripts) — https://github.com/KechrisLab/MSPrep
- **xtable** (Generates the LaTeX code for the reproducibility summary tables)
- **R (>=4.0)** (Runtime environment for all marr/MSPrep evaluation scripts (requires R version 4.0 or later))

## Examples

```
BiocManager::install("marr"); MarrOutput <- Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.01); MarrPlotSamplepairs(MarrOutput)
```

## Evaluation signals

- Reproduced Table 1 percentages match the published values, e.g., BioTech technical 90.08% vs biological 49.85% of metabolites per sample pair at the >70% cutoff
- Tech dataset shows >70% reproducible metabolites across all sample pairs for both the operator and technical layers
- Reproducibility is declared at the conservative FDR level alpha = 0.01 with data-dependent thresholds c_s and c_m chosen via histogram visualization, not defaults
- All 18 saved R objects (*.Rdata) are regenerated from the 18 R_Scripts (6 per dataset) without error
- A higher reproducibility for technical than biological replicates is observed across all three datasets (Tech, BioTech, Bio)

## Limitations

- The goal was benchmarking, not a comprehensive evaluation of pre-processing steps
- True signals may fall in undetectable regions due to overlap between reproducible and irreproducible components, and the lack of separation between components limits detection in some scenarios
- MaRR tends to have high power except for smaller values of the proportion of reproducible metabolites
- The evaluation repository provides no changelog or software version metadata

## Evidence

- [discussion] We applied the MaRR procedure to measure reproducibility both within and across batches using technical replicates for the Tech and BioTech datasets and biological replicates for the BioTech and Bio: "We applied the MaRR procedure to measure reproducibility both within and across batches using technical replicates for the Tech and BioTech datasets and biological replicates for the BioTech and Bio"
- [abstract] a higher level of reproducibility for technical replicates compared to biological replicates across all the three different datasets: "a higher level of reproducibility for technical replicates compared to biological replicates across all the three different datasets"
- [readme] marr measures the reproducibility of features per sample pair and sample pairs per feature in high-dimensional biological replicate experiments.: "`marr` measures the reproducibility of features per sample pair and
sample pairs per feature in high-dimensional biological replicate
experiments."
- [methods] Project IDs for benchmarking datasets: "Project ID: PR000438 (BioTech dataset)"
- [discussion] data-dependent threshold visualization: [evidence span withheld — anchor status: not_found]
