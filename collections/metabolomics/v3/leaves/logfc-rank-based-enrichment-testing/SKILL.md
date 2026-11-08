---
name: logfc-rank-based-enrichment-testing
description: Use when you have two-group differential-expression results for lipids
  (e.g. cancer vs benign tissue) and want to test whether specific lipid classes,
  total chain lengths, or unsaturation patterns are preferentially enriched.
when_to_use_negative:
- The goal is gene/pathway enrichment on transcriptomic or proteomic data — LSEA is
  a lipid-specific enrichment method operating on lipid class, chain-length, and unsaturation
  annotation sets
- The input is raw lipidomics data (Skyline CSV export, numerical matrix, or Metabolomics
  Workbench download) that has not yet been normalized and differentially analyzed;
  QC, normalization, and the univariate contrast must come first
- The analysis is purely multivariate pattern discovery (PCA, PCoA, OPLS(DA)) where
  enrichment of annotation sets is not the question
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: R/Bioconductor package providing lsea (lipid set enrichment analysis ranked
    by logFC) and plot_enrichment for visualizing enriched lipid classes and chain
    lengths
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which lipidr's LipidomicsExperiment objects, lsea,
    and plot_enrichment are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:9ffd99d229f5821970d67633f0e16d6f269f036e396f302d90c8da6fb0a07459
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# logfc-rank-based-enrichment-testing

## Summary

Apply lipidr's novel lipid set enrichment analysis (LSEA) with rank.by="logFC" to detect preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns among differentially regulated lipids, then visualize with plot_enrichment.

## When to use

Use when you have two-group differential-expression results for lipids (e.g. cancer vs benign tissue) and want to test whether specific lipid classes, total chain lengths, or unsaturation patterns are preferentially enriched.

## When NOT to use

- The goal is gene/pathway enrichment on transcriptomic or proteomic data — LSEA is a lipid-specific enrichment method operating on lipid class, chain-length, and unsaturation annotation sets
- The input is raw lipidomics data (Skyline CSV export, numerical matrix, or Metabolomics Workbench download) that has not yet been normalized and differentially analyzed; QC, normalization, and the univariate contrast must come first
- The analysis is purely multivariate pattern discovery (PCA, PCoA, OPLS(DA)) where enrichment of annotation sets is not the question

## Inputs

- lipidr LipidomicsExperiment (extends SummarizedExperiment) with sample annotation/groups loaded via add_sample_annotation
- Two-group differential analysis results (e.g. Cancer vs Benign/Metastasis contrast from the ST001111 lipidomics dataset processed with lipidr)
- Lipid names automatically parsed by lipidr into class, total chain length, and unsaturation annotations

## Outputs

- lsea result object containing significant_lipidsets (lipid sets passing the significance threshold)
- Enrichment plots from plot_enrichment for lipid class annotation sets and total chain-length annotation sets (saved as PNG figures)
- Significant lipid sets table written to lsea_results.csv

## How to apply

Run lsea on the LipidomicsExperiment object (or the differential-analysis result object) with rank.by="logFC" so lipid sets are ranked by log fold change rather than p-value. Extract the significant lipid sets from the returned lsea result object (significant_lipidsets) — these are the annotation sets passing the significance threshold. Render plot_enrichment twice: once on the lsea results for lipid class annotation sets, and once for total chain-length annotation sets. Write the significant lipid sets table to CSV (e.g. lsea_results.csv) and save the class and chain-length enrichment plots as PNG figures. LSEA detects preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns, so choose the annotation set type that answers the biological question at hand.

## Related tools

- **lipidr** (R/Bioconductor package providing lsea (lipid set enrichment analysis ranked by logFC) and plot_enrichment for visualizing enriched lipid classes and chain lengths) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which lipidr's LipidomicsExperiment objects, lsea, and plot_enrichment are executed)

## Examples

```
lsea_results <- lsea(two_group, rank.by = "logFC"); plot_enrichment(lsea_results, annotation = "class"); plot_enrichment(lsea_results, annotation = "totalchain")
```

## Evaluation signals

- The lsea call used rank.by="logFC" (not p-value ranking) before extracting significant sets
- significant_lipidsets from the returned lsea object is non-empty and its members correspond to lipid class or chain-length annotation sets parsed from lipid names
- Two plot_enrichment figures are produced: one for lipid class sets and one for total chain-length sets
- Outputs include lsea_results.csv with the significant lipid sets table and PNG enrichment plots
- Results are reproducible on the ST001111 two-group (Cancer vs Benign/Metastasis) contrast from Metabolomics Workbench

## Limitations

- LSEA detects preferential enrichment only of lipid classes, total chain lengths, or unsaturation patterns — enrichment at other annotation granularities is out of scope
- Univariate comparisons (including the two-group contrast feeding LSEA) require loaded clinical variables; multi-group comparisons and confounder adjustment change the design
- No changelog was found for the tool, so version-specific behavior of lsea/plot_enrichment is not documented here

## Evidence

- [readme] Novel enrichment implemented, README statement: [evidence span withheld — anchor status: external_doc]
