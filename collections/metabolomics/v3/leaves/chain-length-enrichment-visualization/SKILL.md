---
name: chain-length-enrichment-visualization
description: Use when a lipidr differential analysis (e.g. two-group Cancer vs Benign
  contrast) has been ranked by logFC and you need to render and save enrichment plots
  for chain-length or class lipid sets.
when_to_use_negative:
- Input is not a lipidr lsea result (no ranked lipid sets) — plot_enrichment requires
  an lsea object, not raw intensities or a differential statistics table alone
- The analysis goal is detecting chain-unsaturation patterns or other lipid set types
  not among the class/chain-length annotations you intend to plot
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: lipidr
  role: Runs lsea enrichment on logFC-ranked lipids and renders plot_enrichment figures
    for class and chain-length sets
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which lipidr's lsea and plot_enrichment functions are
    executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:f19338bd6315c0f0fe32f63b19a6b300a5f5092f9015f193f5954925685f2bd3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# chain-length-enrichment-visualization

## Summary

Visualizes lipid set enrichment analysis (LSEA) results from lipidr, focusing on the significantly enriched total chain-length (and lipid class) annotation sets in a differential lipidomics contrast.

## When to use

Use when a lipidr differential analysis (e.g. two-group Cancer vs Benign contrast) has been ranked by logFC and you need to render and save enrichment plots for chain-length or class lipid sets.

## When NOT to use

- Input is not a lipidr lsea result (no ranked lipid sets) — plot_enrichment requires an lsea object, not raw intensities or a differential statistics table alone
- The analysis goal is detecting chain-unsaturation patterns or other lipid set types not among the class/chain-length annotations you intend to plot

## Inputs

- lipidr LipidomicsExperiment with a two-group differential analysis contrast (e.g. Cancer-Benign/Metastasis comparison from ST001111)
- lsea result object generated with rank.by='logFC'

## Outputs

- significant_lipidsets table exported as lsea_results.csv
- PNG figure of plot_enrichment for lipid class annotation sets
- PNG figure of plot_enrichment for total chain-length annotation sets

## How to apply

Run lsea() on the LipidomicsExperiment containing the two-group contrast with rank.by='logFC' so sets are ranked by log fold change. Extract the significant_lipidsets component from the returned lsea result object to obtain lipid sets passing the significance threshold. Call plot_enrichment() on the lsea results twice: once for lipid class annotation sets and once for total chain-length annotation sets, saving each as a PNG figure. Write the significant lipid sets table to lsea_results.csv. Correctness is judged by whether the rendered plots highlight the enriched lipid classes and, alternatively, the significantly enriched chain lengths.

## Related tools

- **lipidr** (Runs lsea enrichment on logFC-ranked lipids and renders plot_enrichment figures for class and chain-length sets) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which lipidr's lsea and plot_enrichment functions are executed)

## Examples

```
lsea_res <- lsea(two_group, rank.by="logFC"); significant_lipidsets <- lsea_res$significant_lipidsets; plot_enrichment(lsea_res, annotation="class"); plot_enrichment(lsea_res, annotation="total_chain")
```

## Evaluation signals

- lsea result object contains a significant_lipidsets component with lipid sets passing the significance threshold
- Ranking used rank.by='logFC', so enrichment statistics derive from log fold changes of the two-group contrast
- Two PNG figures are produced: one for lipid class sets and one for total chain-length sets
- Plots visually highlight enriched lipid classes and, alternatively, significantly enriched chain lengths
- lsea_results.csv is written and contains the significant lipid sets table

## Limitations

- LSEA in lipidr detects preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns; chain-length visualization is only meaningful for such annotated sets
- The skill depends on a completed two-group differential analysis; without a valid contrast and logFC ranking, lsea cannot rank sets
- No changelog found for lipidr, so function behavior across versions may vary

## Evidence

- [intro] A novel lipid set enrichment analysis (LSEA) is implemented to detect preferential enrichment of certain lipid classes, total chain lengths or unsaturation patterns.: "A novel lipid set enrichment analysis (LSEA) is implemented to detect preferential enrichment of certain lipid classes, total chain lengths or unsaturation patterns."
- [readme] Plots for visualization of enrichment results are also implemented.: [evidence span withheld — anchor status: external_doc]
- [readme] Univariate analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots.: "Univariate analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots."
