---
name: metabolomics-normalization-pipeline-comparison
description: Use when replicate-containing metabolomics data have missing values and
  you must choose between imputation methods (BPCA, kNN, RF-median, RF-quantile) crossed
  with normalizations (quantile, median, RUV), scored by reproducibility rather than
  CV/RSD.
when_to_use_negative:
- When the goal is a comprehensive evaluation of pre-processing steps — the authors
  state their goal was not comprehensive evaluation, but only to show how MaRR can
  be used as a benchmark for comparison.
- When the data lack technical or biological replicate samples, since MaRR reproducibility
  is defined across replicate sample pairs.
- When reproducibility filtering by relative standard deviation (CV/RSD) is expected
  to suffice — a 25% RSD cutoff removed none of the MaRR-identified irreproducible
  metabolites in the BioTech dataset.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSPrep
  role: Pre-processes the metabolomics data under the 6 imputation-by-normalization
    combinations (BPCA/kNN x quantile/median/RUV); installed from GitHub ref 'feature-generalize'
  repo: https://github.com/KechrisLab/MSPrep
- name: marr
  role: Runs the MaRR nonparametric maximal-rank procedure to quantify reproducibility
    of features per sample pair and sample pairs per feature for each pipeline
  repo: http://bioconductor.org/packages/marr
- name: BiocManager
  role: Installs the stable Bioconductor release of marr (requires R >= 4.0)
- name: devtools
  role: Installs MSPrep from GitHub via install_github
- name: xtable
  role: Generates the LaTeX code of the pipeline-comparison table in the same order
    as the tables in MaRR_real_results.pdf
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:d8fb74f90e1de02a6fc76cbdd195d40efc886e2fb77d65233937478bca285650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KechrisLab/MSPrep
  tools:
  - BiocManager
  - MSPrep
  - R
  - devtools
  - marr
  - xtable
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# metabolomics-normalization-pipeline-comparison

## Summary

Benchmarks missing-value imputation and normalization pipelines for MS-metabolomics data by running the MaRR (Maximum Rank Reproducibility) procedure on each preprocessed dataset and comparing the percentage of reproducible metabolites per sample pair and sample pairs per metabolite.

## When to use

Use when replicate-containing metabolomics data have missing values and you must choose between imputation methods (BPCA, kNN, RF-median, RF-quantile) crossed with normalizations (quantile, median, RUV), scored by reproducibility rather than CV/RSD.

## When NOT to use

- When the goal is a comprehensive evaluation of pre-processing steps — the authors state their goal was not comprehensive evaluation, but only to show how MaRR can be used as a benchmark for comparison.
- When the data lack technical or biological replicate samples, since MaRR reproducibility is defined across replicate sample pairs.
- When reproducibility filtering by relative standard deviation (CV/RSD) is expected to suffice — a 25% RSD cutoff removed none of the MaRR-identified irreproducible metabolites in the BioTech dataset.

## Inputs

- Raw MS-metabolomics replicate dataset (e.g., BioTech dataset, Project ID PR000438 from Metabolomics Workbench)
- Six pre-processed data objects (BPCA/kNN imputation crossed with quantile/median/RUV normalization)
- Saved R objects from the 'saved_objects' folder of the Real_Data repository

## Outputs

- Percentage of reproducible metabolites per sample pair and reproducible sample pairs per metabolite for each of the 6 pipelines
- Aggregated comparison table (LaTeX code via xtable, ordered as in MaRR_real_results.pdf)
- Intermediate saved R objects per pipeline

## How to apply

Download the replicate dataset (e.g., the BioTech dataset, Metabolomics Workbench Project ID PR000438) and preprocess it under all 6 combinations of imputation (BPCA or kNN) crossed with normalization (quantile, median, or RUV) using MSPrep installed from GitHub (KechrisLab/MSPrep, ref 'feature-generalize') under R (>=4.0). For each of the 6 processed data objects, run the MaRR nonparametric maximal-rank procedure via the marr R/Bioconductor package (Marr() with pSamplepairs and pFeatures thresholds, default 0.75, alpha 0.05) to estimate reproducibility of metabolites per sample pair and of sample pairs per metabolite, saving intermediate R objects. Aggregate the percentages across the 6 pipelines and generate LaTeX table code with the xtable package in the same order as the tables in MaRR_real_results.pdf. Judge correctness by checking that RF-median or RF-quantile preprocessing achieves maximum reproducibility, outperforming BPCA- and kNN-based combinations, and that the percentage of reproducible metabolites per sample pair for technical replicates exceeds 70% and is much higher than for biological replicates.

## Related tools

- **MSPrep** (Pre-processes the metabolomics data under the 6 imputation-by-normalization combinations (BPCA/kNN x quantile/median/RUV); installed from GitHub ref 'feature-generalize') — https://github.com/KechrisLab/MSPrep
- **marr** (Runs the MaRR nonparametric maximal-rank procedure to quantify reproducibility of features per sample pair and sample pairs per feature for each pipeline) — http://bioconductor.org/packages/marr
- **BiocManager** (Installs the stable Bioconductor release of marr (requires R >= 4.0))
- **devtools** (Installs MSPrep from GitHub via install_github)
- **xtable** (Generates the LaTeX code of the pipeline-comparison table in the same order as the tables in MaRR_real_results.pdf)

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.05)
```

## Evaluation signals

- For the BioTech dataset, RF-median or RF-quantile pre-processing achieves maximum reproducibility, outperforming BPCA- and kNN-based combinations (Table 2).
- The tabulated percentage of reproducible metabolites per sample pair for technical replicates exceeds 70% and is much higher than for biological replicates.
- All 6 pipelines are evaluated in both directions: reproducible metabolites per sample pair and reproducible sample pairs per metabolite.
- Generated LaTeX table order matches the tables in MaRR_real_results.pdf from the 'Output' folder of the Real_Data repository.
- As a downstream sanity check, a 25% RSD cutoff must fail to remove the MaRR-identified irreproducible metabolites (Table S4, Figs. S24–S25).

## Limitations

- The comparison is a benchmark of pipelines, not a comprehensive evaluation of pre-processing steps.
- True signals may fall in undetectable regions because of overlap between the reproducible and irreproducible components, limiting detection.
- MaRR tends to have high power except for smaller values of the proportion of reproducible metabolites.

## Evidence

- [results] For the BioTech dataset, processing the data using RF-median or RF-quantile achieved the maximum reproducibility (Table 2), outperforming BPCA- and kNN-based pre-processing combinations.: [evidence span withheld — anchor status: not_found]
- [methods] Pipeline grid definition: "Imputation- BPCA and kNN and Normalization- quantile, median and RUV"
- [methods] MSPrep installation: "install_github("KechrisLab/MSPrep",ref="feature-generalize")"
- [discussion] MaRR applicability to preprocessing comparisons: "the MaRR procedure can also be employed to evaluate different pre-processing pipelines (e.g., missing value imputation, normalization)."
- [readme] marr package purpose: "marr measures the reproducibility of features per sample pair and sample pairs per feature in high-dimensional biological replicate experiments."
- [readme] marr installation requirements: "The stable release version can be installed using the Bioconductor installer as follows. Note that installation requires R version 4.0 or later."
