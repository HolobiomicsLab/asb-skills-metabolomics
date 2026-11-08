---
name: fdr-multiple-testing-correction
description: Use when many p-values are generated per factor–pair hypothesis (GEE,
  Spearman correlation, Mann–Whitney/Kruskal–Wallis cluster comparisons, or EWAS atlas
  over-representation tests) and an FDR-adjusted p-value column must be reported for
  each tested pair.
when_to_use_negative:
- Only one hypothesis is tested (e.g. a single SNF cluster comparison) — correction
  is unnecessary and deflates power
- The p-values come from the metabolomics QC/filtering or MOFA/SNF model-building
  steps rather than from association or enrichment testing — there is no family of
  hypotheses to correct
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Nextflow
  role: Workflow manager that orchestrates the association-analysis and EWAS-enrichment
    modules whose p-values are FDR-corrected and whose tables are written via --output
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: R (MOFA2 / gee ecosystem)
  role: R/Bioconductor environment in which MOFA2 (1.3.4) factor weights, GEE models,
    and the multiple-testing correction are computed
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:5c085f5a5164ae00b641f0f980ee5d9e6bb92c91e2ace44f07c4c6adadc8d260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - MOFA2
  - Nextflow
  - R (MOFA2 / gee ecosystem)
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# FDR multiple-testing correction

## Summary

Corrects raw p-values for multiple testing (false discovery rate) across all tested factor–dimension and factor–trait pairs in the ACTIONdemonstrator workflow, covering GEE/association tests on MOFA factors and SNF clusters against MCA behavioral dimensions and EWAS atlas enrichment tests on top-100 CpG sets.

## When to use

Use when many p-values are generated per factor–pair hypothesis (GEE, Spearman correlation, Mann–Whitney/Kruskal–Wallis cluster comparisons, or EWAS atlas over-representation tests) and an FDR-adjusted p-value column must be reported for each tested pair.

## When NOT to use

- Only one hypothesis is tested (e.g. a single SNF cluster comparison) — correction is unnecessary and deflates power
- The p-values come from the metabolomics QC/filtering or MOFA/SNF model-building steps rather than from association or enrichment testing — there is no family of hypotheses to correct

## Inputs

- Raw p-values from GEE association models of MOFA factors/SNF clusters vs. MCA behavioral dimensions (NTR)
- Raw p-values from Mann–Whitney U, Kruskal–Wallis, and Spearman rank correlation tests (LUMC-Curium)
- Nominal hypergeometric / Fisher's exact enrichment p-values and overlap sizes for each MOFA factor–EWAS atlas trait pair

## Outputs

- association_statistics.csv with one row per tested factor–dimension pair per cohort, including raw and FDR-corrected p-values
- Factor × trait EWAS enrichment table with overlap-size and FDR-adjusted significance columns in the Nextflow --output directory

## How to apply

Collect raw p-values from each association module: for the NTR cohort, p-values from GEE models (R package 'gee', exchangeable working correlation, Gaussian link, up to 100 iterations) regressing each MCA behavioral dimension score on each MOFA omics factor and SNF cluster membership; for LUMC-Curium, p-values from Mann–Whitney U tests (2 clusters), Kruskal–Wallis tests (>2 clusters), and Spearman rank correlations. For the EWAS atlas annotation step, collect hypergeometric/Fisher's exact p-values from over-representation testing of each methylation factor's top-100 weighted CpG set against trait-specific CpG sets. Apply FDR correction across all tested factor–dimension (or factor–trait) combinations within each analysis module and both cohorts. Emit one row per tested pair in association_statistics.csv listing cohort, predictor, behavioral dimension, test type, test statistic, and both raw and FDR-corrected p-values; for the enrichment step, populate overlap-size and significance columns in the factor × trait table written to the Nextflow --output directory.

## Related tools

- **Nextflow** (Workflow manager that orchestrates the association-analysis and EWAS-enrichment modules whose p-values are FDR-corrected and whose tables are written via --output) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **R (MOFA2 / gee ecosystem)** (R/Bioconductor environment in which MOFA2 (1.3.4) factor weights, GEE models, and the multiple-testing correction are computed) — https://github.com/Xomics/ACTIONdemonstrator_workflow

## Examples

```
Rscript -e 'assoc <- read.csv("association_statistics.csv"); assoc$fdr_p <- stats::p.adjust(assoc$p_value, method = "BH")'
```

## Evaluation signals

- association_statistics.csv contains exactly one row per tested factor–dimension combination for both cohorts
- Every row includes a populated FDR-corrected p-value alongside the raw p-value
- The test type recorded per row matches the cohort: GEE with exchangeable correlation for NTR; Mann–Whitney/Kruskal–Wallis for LUMC-Curium
- The enrichment table has one entry per factor–trait pair with populated overlap-size and significance fields
- FDR-corrected p-values are always ≥ their corresponding raw p-values and lie within [0, 1]

## Limitations

- Privacy regulations prohibit sharing personally identifiable omics/clinical data, so only a synthetic dataset is provided for validating the correction step
- GEE models may require up to 100 iterations to converge for the NTR twin-family structure; non-convergent models can leave pairs untested before correction
- FDR correction is applied across tested pairs within a module; it does not replace cohort-specific handling of relatedness (GEE) or cluster-number choice (eigengap method in SNFpy)

## Evidence

- [other] Correct all association p-values for multiple testing across the tested factor–dimension pairs using false discovery rate (FDR) correction in both cohorts.: [evidence span withheld — anchor status: not_found]
- [other] Correct the enrichment p-values for multiple testing across all factor-trait combinations (FDR).: [evidence span withheld — anchor status: not_found]
- [discussion] We determined correlations among the obtained factors capturing the omics and behavioral data, respectively, using Spearman's rank correlation: [evidence span withheld — anchor status: not_found]
- [discussion] We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
