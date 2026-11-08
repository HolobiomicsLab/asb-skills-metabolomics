---
name: lipid-class-annotation-visualization
description: Use when you have lsea enrichment results (e.g., ranked by logFC from
  a two-group lipidomics contrast) and need to render class-level or chain-length
  enrichment plots.
when_to_use_negative:
- The input is raw lipid intensity matrices or Skyline exports that have not yet been
  differentially analyzed — LSEA ranking requires per-lipid statistics such as logFC.
- The goal is generic gene-set enrichment (GSEA) on non-lipid features; lipidr's LSEA
  is specific to lipid classes, total chain lengths, or unsaturation patterns inferred
  from lipid names.
- You need multivariate pattern discovery (PCA, PCoA, OPLS-DA) rather than enrichment
  of annotation sets among regulated lipids.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: lipidr
  role: R/Bioconductor package providing lsea for lipid set enrichment and plot_enrichment
    for rendering class and chain-length enrichment plots
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which lipidr's LipidomicsExperiment, lsea, and plotting
    functions are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:f38827247d59627589cb3407ff7bc31737f43418043311ab608242b4b21e0258
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lipid-class-annotation-visualization

## Summary

Visualize lipid set enrichment analysis (LSEA) results in lipidr with plot_enrichment, highlighting lipid classes and total chain-length annotation sets that are preferentially enriched among differentially regulated lipids.

## When to use

Use when you have lsea enrichment results (e.g., ranked by logFC from a two-group lipidomics contrast) and need to render class-level or chain-length enrichment plots.

## When NOT to use

- The input is raw lipid intensity matrices or Skyline exports that have not yet been differentially analyzed — LSEA ranking requires per-lipid statistics such as logFC.
- The goal is generic gene-set enrichment (GSEA) on non-lipid features; lipidr's LSEA is specific to lipid classes, total chain lengths, or unsaturation patterns inferred from lipid names.
- You need multivariate pattern discovery (PCA, PCoA, OPLS-DA) rather than enrichment of annotation sets among regulated lipids.

## Inputs

- lsea result object from lipidr's lipid set enrichment analysis (e.g., run with rank.by='logFC')
- LipidomicsExperiment with differential analysis results for a two-group contrast (e.g., Cancer vs Benign/Metastasis from ST001111)
- Lipid names parsed by lipidr into lipid class and total chain-length annotation sets

## Outputs

- Enrichment plot (PNG) for lipid class annotation sets via plot_enrichment
- Enrichment plot (PNG) for total chain-length annotation sets via plot_enrichment
- Table of significant lipid sets (e.g., significant_lipidsets from the lsea result) written to lsea_results.csv

## How to apply

After running lsea on a LipidomicsExperiment differential-analysis result (e.g., rank.by='logFC' for a Cancer vs Benign/Metastasis contrast in the ST001111 dataset), extract significant_lipidsets from the returned lsea result to obtain lipid sets passing the significance threshold. Call plot_enrichment on the lsea results once for lipid class annotation sets and once for total chain-length annotation sets. lipidr automatically infers lipid class and chain information from lipid names at import, so the annotation sets are derived from the lipid nomenclature itself. Persist the significant lipid sets table (e.g., lsea_results.csv) and save each plot as a PNG figure. Correct application is judged by whether the plots highlight the enriched lipid classes and, alternatively, the significantly enriched chain lengths consistent with the significant_lipidsets table.

## Related tools

- **lipidr** (R/Bioconductor package providing lsea for lipid set enrichment and plot_enrichment for rendering class and chain-length enrichment plots) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which lipidr's LipidomicsExperiment, lsea, and plotting functions are executed)

## Examples

```
lsea_res <- lsea(two_group, rank.by = "logFC"); plot_enrichment(lsea_res, annotation = "class"); plot_enrichment(lsea_res, annotation = "totalchain")
```

## Evaluation signals

- significant_lipidsets extracted from the lsea result is non-empty and matches the lipid sets highlighted in the plot_enrichment output
- Two PNG figures are produced: one for lipid class annotation sets and one for total chain-length annotation sets
- lsea_results.csv contains the significant lipid sets table with enrichment statistics
- Ranking used for lsea is logFC from the two-group (e.g., Cancer-Benign/Metastasis) differential contrast, not raw intensities
- Enriched classes/chain lengths in the plots correspond to lipids preferentially regulated between the two tissue groups

## Limitations

- LSEA detects preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns — it does not identify individual regulated lipid species directly.
- Annotation sets depend on lipidr's automatic inference of lipid class and chain information from lipid names; non-standard lipid nomenclature may break set construction.
- Interpretation is limited to the contrast and ranking metric supplied to lsea (e.g., logFC); results may differ under other rank.by choices.

## Evidence
