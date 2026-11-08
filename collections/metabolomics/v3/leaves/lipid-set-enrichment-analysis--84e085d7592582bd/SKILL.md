---
name: lipid-set-enrichment-analysis
description: Use when a lipidomics dataset (Skyline export, numerical matrix, or Metabolomics
  Workbench file) has been differentially analyzed with lipidr and you need to know
  which lipid classes or chain characteristics are enriched in the regulated lipids.
when_to_use_negative:
- Input lipids are not annotated with class/chain information — LSEA depends on lipidr
  automatically inferring lipid class and chain information from lipid names
- The goal is gene/protein set enrichment rather than lipid-annotation-set enrichment
  — LSEA tests lipid classes, chain lengths or unsaturation patterns, not gene sets
- No differential or ranking statistics exist yet — lsea requires ranked lipids (e.g.
  by logFC) from a prior differential analysis
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: R/Bioconductor package providing lsea() for lipid set enrichment and plot_enrichment()
    for visualizing class and chain-length enrichment
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment; lipidr is an open-source R/Bioconductor package and LipidomicsExperiment
    extends SummarizedExperiment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:41a1351c34d7ed3ee42914a32f94b89105d50881fb91045b7b94ad74907a33d9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADViSELipidomics
  - R/Bioconductor
  - lipidr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lipid-set-enrichment-analysis

## Summary

Run lipidr's novel Lipid Set Enrichment Analysis (LSEA) to detect preferential enrichment of lipid classes, total chain lengths or unsaturation patterns among ranked differentially expressed lipids, and visualize the enrichment results.

## When to use

Use when a lipidomics dataset (Skyline export, numerical matrix, or Metabolomics Workbench file) has been differentially analyzed with lipidr and you need to know which lipid classes or chain characteristics are enriched in the regulated lipids.

## When NOT to use

- Input lipids are not annotated with class/chain information — LSEA depends on lipidr automatically inferring lipid class and chain information from lipid names
- The goal is gene/protein set enrichment rather than lipid-annotation-set enrichment — LSEA tests lipid classes, chain lengths or unsaturation patterns, not gene sets
- No differential or ranking statistics exist yet — lsea requires ranked lipids (e.g. by logFC) from a prior differential analysis

## Inputs

- lipidr differential analysis results object from a two-group contrast (e.g. Cancer-Benign/Metastasis comparison of ST001111)
- LipidomicsExperiment built from Skyline-exported CSV, numerical matrix + sample annotation table, or Metabolomics Workbench files
- Ranking statistic for lipids, e.g. log fold change (rank.by="logFC")

## Outputs

- lsea result object containing significant_lipidsets (lipid sets passing the significance threshold)
- Significant lipid sets table exported as CSV (e.g. lsea_results.csv)
- Enrichment plots (PNG) for lipid class annotation sets and total chain-length annotation sets

## How to apply

First complete a lipidr differential analysis (e.g. a two-group Cancer vs Benign comparison on the ST001111 study processed through lipidr, whose inputs are Skyline CSV exports or numerical matrices plus a sample-annotation table). Rank the lipids by an effect-size statistic — for the two-group contrast use rank.by="logFC" — and run lsea() on the differential results object; LSEA tests lipid annotation sets (classes, total chain lengths, unsaturation patterns) rather than gene sets. Extract the significant_lipidsets from the returned lsea result object to identify sets passing the significance threshold. Render plot_enrichment on the lsea results, once with lipid class annotation sets and once with total chain-length annotation sets, then export the significant sets table (e.g. lsea_results.csv) and the plots as PNG figures.

## Related tools

- **lipidr** (R/Bioconductor package providing lsea() for lipid set enrichment and plot_enrichment() for visualizing class and chain-length enrichment) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment; lipidr is an open-source R/Bioconductor package and LipidomicsExperiment extends SummarizedExperiment)

## Examples

```
significant_lipidsets <- lsea(two_group, rank.by="logFC")
plot_enrichment(lsea_results, annotation="class")
plot_enrichment(lsea_results, annotation="total_chain")
```

## Evaluation signals

- significant_lipidsets extracted from the lsea result is non-empty and contains lipid classes and/or total chain-length sets consistent with the ranked logFC direction
- plot_enrichment renders successfully for both lipid class annotation sets and total chain-length annotation sets
- lsea_results.csv contains the significant sets table with entries matching the extracted significant_lipidsets
- Class and chain-length enrichment plots are saved as PNG figures without errors
- The lsea call reports enrichment in the expected direction for the Cancer vs Benign/Metastasis contrast on ST001111

## Limitations

- LSEA detects enrichment only of predefined lipid annotation sets — classes, total chain lengths or unsaturation patterns — so lipids with unparseable names cannot be assigned to sets
- Enrichment is driven by the ranking statistic; a two-group analysis ranked by logFC will not account for statistical significance weights unless another rank.by option is chosen
- Interpretation of enrichment relies on the differential contrast being defined correctly (clinical variables loaded via add_sample_annotation with matching sample names)

## Evidence

- [readme] A novel lipid set enrichment analysis is implemented to detect preferential regulation of certain lipid classes, total chain lengths or unsaturation patterns.: [evidence span withheld — anchor status: external_doc]
