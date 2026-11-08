---
name: differential-lipidomics-results-interpretation
description: Use when you have a two-group (or multi-group) differential lipidomics
  comparison and want to know which lipid classes or chain attributes are preferentially
  enriched among the regulated lipids.
when_to_use_negative:
- You need gene-set style enrichment on generic feature IDs — LSEA operates on lipid
  annotation sets (classes, total chain lengths, unsaturation patterns), not arbitrary
  feature sets
- You have only raw intensities with no differential ranking (e.g. logFC) yet — LSEA
  requires prior univariate/differential analysis of clinical variables
- The goal is quality control or normalization rather than interpretation of regulated
  lipids — lipidr's QC/plotting and normalization steps serve that purpose instead
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: lipidr
  role: R/Bioconductor package providing lsea() for the lipid set enrichment analysis
    and plot_enrichment() for visualizing enriched lipid classes and chain lengths
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which the lipidr LSEA workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:1b4051ec33e4b5ede00aafb032c562a21ec0f87f93fc09e10edf3558e48d20b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# differential-lipidomics-results-interpretation

## Summary

Interpret differential lipidomics results by running lipidr's novel Lipid Set Enrichment Analysis (LSEA), which ranks differentially regulated lipids (e.g. by logFC) to detect preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns, and visualizes the enriched sets.

## When to use

Use when you have a two-group (or multi-group) differential lipidomics comparison and want to know which lipid classes or chain attributes are preferentially enriched among the regulated lipids.

## When NOT to use

- You need gene-set style enrichment on generic feature IDs — LSEA operates on lipid annotation sets (classes, total chain lengths, unsaturation patterns), not arbitrary feature sets
- You have only raw intensities with no differential ranking (e.g. logFC) yet — LSEA requires prior univariate/differential analysis of clinical variables
- The goal is quality control or normalization rather than interpretation of regulated lipids — lipidr's QC/plotting and normalization steps serve that purpose instead

## Inputs

- lipidr differential analysis results object for a group contrast (e.g. two-group Cancer vs Benign comparison), with lipids ranked by log fold change
- LipidomicsExperiment from lipidr (imported from a numerical matrix, Skyline CSV export, or Metabolomics Workbench files) with sample annotations

## Outputs

- Significant lipid sets table (e.g. lsea_results.csv) listing lipid sets passing the LSEA significance threshold
- plot_enrichment figures for lipid class annotation sets (PNG)
- plot_enrichment figures for total chain-length annotation sets (PNG)

## How to apply

Take the differential analysis results produced by lipidr's univariate workflow (ranked by a statistic such as logFC from the chosen contrast, e.g. Cancer vs Benign/Metastasis). Call lsea() on the differential results with rank.by="logFC" to detect preferential enrichment of lipid annotation sets (classes, total chain lengths, unsaturation patterns), then extract the significant_lipidsets from the returned result object. Render plot_enrichment() on the lsea results twice: once for lipid class annotation sets and once for total chain-length annotation sets. Finally, persist the significant lipid sets table (e.g. lsea_results.csv) and save the class and chain-length enrichment plots as PNG figures; correct application is judged by the enrichment plots highlighting biologically coherent lipid classes/chain lengths consistent with the logFC ranking.

## Related tools

- **lipidr** (R/Bioconductor package providing lsea() for the lipid set enrichment analysis and plot_enrichment() for visualizing enriched lipid classes and chain lengths) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which the lipidr LSEA workflow is executed)

## Examples

```
lsea_results <- lsea(two_group, rank.by = "logFC"); significant_lipidsets <- lsea_results$significant_lipidsets; plot_enrichment(lsea_results)
```

## Evaluation signals

- lsea() is run with rank.by="logFC" on the differential results object for the chosen contrast (e.g. Cancer-Benign/Metastasis)
- significant_lipidsets extracted from the lsea result object is non-empty and matches the sets passing the significance threshold written to lsea_results.csv
- plot_enrichment renders without error on both lipid class annotation sets and total chain-length annotation sets
- Enrichment plots highlight lipid classes/chain lengths whose member lipids show consistent logFC direction in the differential analysis
- Output files (lsea_results.csv, class and chain-length PNG figures) are produced with the expected schemas

## Limitations

- LSEA detects preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns only — other lipid attributes are not covered
- Results depend on the quality of the upstream differential analysis and the ranking statistic; multi-group contrasts and confounder adjustment must be handled in the univariate step
- lipidr expects consistent sample names between the data matrix and sample annotation table when building the LipidomicsExperiment

## Evidence

- [intro] A novel lipid set enrichment analysis (LSEA) is implemented to detect preferential enrichment of certain lipid classes, total chain lengths or unsaturation patterns.: "A novel lipid set enrichment analysis (LSEA) is implemented to detect preferential enrichment of certain lipid classes, total chain lengths or unsaturation patterns."
- [readme] A novel lipid set enrichment analysis is implemented to detect preferential regulation of certain lipid classes, total chain lengths or unsaturation patterns.: [evidence span withheld — anchor status: external_doc]
