---
name: rsd-computation-replicate-metabolomics
description: Use when a replicate-structure metabolomics dataset needs a per-metabolite
  variability score (RSD/CV) across biological or technical replicates, or when evaluating
  whether an RSD filter removes irreproducible features.
when_to_use_negative:
- The goal is not to benchmark a variability filter but to fully compare pre-processing
  pipelines — the paper's stated goal 'was not to do a comprehensive evaluation of
  pre-processing steps, but instead to show how this measure can be used as a benchmark
  for comparison'
- Only a single sample per condition is available, so no replicate-based variability
  (RSD/CV) can be computed
- A reproducibility assessment is needed that does not rely on parametric or fixed-threshold
  assumptions — prefer the MaRR maximal rank statistic itself
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R (>=4.0)
  role: Analysis environment for computing per-metabolite RSD across biological and
    technical replicates and generating the comparison scatter plots and tables
- name: marr
  role: Provides the MaRR maximal-rank reproducibility labels (75% sample-pair threshold)
    against which the RSD cutoff is benchmarked
  repo: https://github.com/Ghoshlab/marr
- name: xtable
  role: Generates the LaTeX comparison table (Table S4 analogue) summarizing RSD-cutoff
    removal of MaRR-irreproducible metabolites
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:3de2c36c0623eb08e0a1a44e86b84e5f96933cbf5f4640b5b1e202c40536e9fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ghoshlab/marr
  tools:
  - R
  - R (>=4.0)
  - marr
  - xtable
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# rsd-computation-replicate-metabolomics

## Summary

Computes the relative standard deviation (RSD, %) of metabolite abundances across biological and technical replicates and benchmarks this conventional variability measure against MaRR-based reproducibility labels, showing whether a fixed RSD cutoff (e.g. 25%) can separate reproducible from irreproducible metabolites.

## When to use

Use when a replicate-structure metabolomics dataset needs a per-metabolite variability score (RSD/CV) across biological or technical replicates, or when evaluating whether an RSD filter removes irreproducible features.

## When NOT to use

- The goal is not to benchmark a variability filter but to fully compare pre-processing pipelines — the paper's stated goal 'was not to do a comprehensive evaluation of pre-processing steps, but instead to show how this measure can be used as a benchmark for comparison'
- Only a single sample per condition is available, so no replicate-based variability (RSD/CV) can be computed
- A reproducibility assessment is needed that does not rely on parametric or fixed-threshold assumptions — prefer the MaRR maximal rank statistic itself

## Inputs

- Pre-processed metabolite-by-sample abundance matrix with biological and technical replicate annotations (e.g. RF-quantile pre-processed BioTech dataset from saved R objects in 'saved_objects')
- Raw data from Metabolomics Workbench Project ID PR000438 (BioTech dataset) or PR000909/PR000907 (Bio dataset)
- MaRR reproducibility labels per metabolite (from marr package or source_MaRR.R)

## Outputs

- Per-metabolite RSD values across biological replicates and across technical replicates
- Scatter plots of MaRR reproducibility versus RSD for biological and technical replicate layers with a 25% RSD cutoff overlaid (Additional file 1: Figs. S24–S25 analogues)
- LaTeX comparison table (xtable) quantifying removal of MaRR-irreproducible metabolites by the RSD cutoff (Table S4 analogue), saved to the 'Output' folder

## How to apply

Load the pre-processed metabolite-by-sample matrix (e.g. the RF-quantile pre-processed BioTech dataset from Metabolomics Workbench Project ID PR000438, imputation plus quantile normalization) with replicates identified. For every metabolite, compute RSD as the standard deviation divided by the mean of abundances across biological replicates, and repeat across technical replicates, expressed as a percentage. Then run the MaRR procedure (marr package, source_MaRR.R from the 'src' folder of marr_paper_evaluations) with a 75% sample-pair reproducibility threshold to label each metabolite reproducible or irreproducible. Plot MaRR reproducibility versus RSD for the biological and technical replicate layers, overlay the conventional 25% RSD cutoff, and tabulate (via xtable) how many MaRR-irreproducible metabolites the cutoff removes. In the BioTech dataset this comparison demonstrates the cutoff's failure, motivating MaRR's data-driven, nonparametric filtering instead of fixed CV/RSD thresholds.

## Related tools

- **R (>=4.0)** (Analysis environment for computing per-metabolite RSD across biological and technical replicates and generating the comparison scatter plots and tables)
- **marr** (Provides the MaRR maximal-rank reproducibility labels (75% sample-pair threshold) against which the RSD cutoff is benchmarked) — https://github.com/Ghoshlab/marr
- **xtable** (Generates the LaTeX comparison table (Table S4 analogue) summarizing RSD-cutoff removal of MaRR-irreproducible metabolites)

## Examples

```
rsd <- 100 * apply(tech[, tech_replicates], 1, function(x) sd(x, na.rm=TRUE)/mean(x, na.rm=TRUE)); Marr(object = dataSE, pSamplepairs=0.75, pFeatures=0.75, alpha=0.05)
```

## Evaluation signals

- Scatter plots must match the qualitative pattern of Additional file 1: Figs. S24–S25 (MaRR reproducibility vs RSD for biological and technical replicates)
- Table S4 must confirm that the 25% RSD cutoff removes none of the MaRR-irreproducible metabolites in the BioTech dataset
- RSD values must be computed separately for the biological replicate layer and the technical replicate layer, not pooled
- Per-metabolite RSD must be finite for metabolites with non-zero mean abundance and reflect percentage units (sd/mean × 100)
- Output plots and LaTeX table must be written to the 'Output' folder in the same table order as MaRR_real_results.pdf

## Limitations

- A conventional 25% RSD filtering cutoff does not remove any MaRR-identified irreproducible metabolites in the BioTech dataset — CV/RSD fails to separate reproducible from irreproducible features
- Reproducible and irreproducible components may lack separation, so real signals 'might fall in the undetectable regions because of overlapping between the reproducible and irreproducible components'
- MaRR power drops for smaller values of the proportion of reproducible metabolites, which affects the labels the RSD cutoff is judged against

## Evidence

- [intro] CV/RSD as the conventional but limited reproducibility measure: "There are few approaches that specifically examine reproducibility for MS-based metabolomics experiments. The most common approach is calculating the coefficient of variation (CV) across replicates."
- [results] RSD cutoff failure finding: [evidence span withheld — anchor status: not_found]
- [discussion] MaRR data-driven filtering as the alternative to fixed thresholds: "our MaRR based data-driven filtering approach has the ability to remove irreproducible metabolites and sample pairs while retaining reproducible metabolites and sample pairs based on simulations."
- [readme] marr measures reproducibility of features per sample pair and sample pairs per feature: "`marr` measures the reproducibility of features per sample pair and sample pairs per feature in high-dimensional biological replicate experiments."
- [discussion] Reproducibility comparison via pairwise replicate layers: "We applied the MaRR procedure to measure reproducibility both within and across batches using technical replicates for the Tech and BioTech datasets and biological replicates for the BioTech and Bio"
