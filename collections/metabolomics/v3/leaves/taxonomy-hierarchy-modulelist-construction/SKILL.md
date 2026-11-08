---
name: taxonomy-hierarchy-modulelist-construction
description: Use when you have a species-level abundance table (e.g. skin microbiome
  relative abundances) plus sample metadata groups, and want enrichment at a custom
  taxonomic level (genus) rather than KEGG pathway/module enrichment.
when_to_use_negative:
- The goal is standard KEGG pathway or module enrichment of KO/compound abundance
  data — use the built-in KOlist/CPDlist databases instead of a custom taxonomy modulelist
- The abundance table has been prefiltered to background features, violating the threshold-free
  GRSA assumption
- There is no grouping variable or metadata to define the contrast (GRSA supports
  multi-group, longitudinal, and continuous groupings, but a comparison is required)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: ReporterScore
  role: R package providing custom_modulelist(), ko.test(), reporter_score()/GRSA(),
    and plot_report() for taxonomy-level enrichment via a custom genus-species module
    list
  repo: https://github.com/Asa12138/ReporterScore
- name: devtools
  role: Used to install the GitHub development versions of ReporterScore and pcutils
- name: pcutils
  role: Dependency installed from GitHub for ReporterScore visualization support
  repo: github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:82499dede2322957d043aa1320b81be862cb51750c156b4f896b094b572cf14c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# taxonomy-hierarchy-modulelist-construction

## Summary

Constructs a custom genus-to-species hierarchical module list with ReporterScore's custom_modulelist() so that GRSA can aggregate species-level abundance statistics into genus-level reporter scores, extending functional enrichment to taxonomy-level enrichment as demonstrated on the IHSMGC skin microbiome dataset.

## When to use

Use when you have a species-level abundance table (e.g. skin microbiome relative abundances) plus sample metadata groups, and want enrichment at a custom taxonomic level (genus) rather than KEGG pathway/module enrichment.

## When NOT to use

- The goal is standard KEGG pathway or module enrichment of KO/compound abundance data — use the built-in KOlist/CPDlist databases instead of a custom taxonomy modulelist
- The abundance table has been prefiltered to background features, violating the threshold-free GRSA assumption
- There is no grouping variable or metadata to define the contrast (GRSA supports multi-group, longitudinal, and continuous groupings, but a comparison is required)

## Inputs

- Species-level abundance table (rownames = species, colnames = samples; counts or relative abundance, unfiltered)
- Sample metadata with grouping variable (e.g. M-/C-cutotype categories), rownames matching sample names
- Genus-to-species mapping and genus-to-description mapping for custom_modulelist()

## Outputs

- Genus-level reporter score / enrichment result table (e.g. grsa_taxonomy_cutotype_results.csv) with adjusted p-values
- Visualization of enriched genera (e.g. plot_report())

## How to apply

Load the species-level abundance table (analogous to the packaged KO_abundance example) and the sample metadata (e.g. cutotype groups) in R; metadata rownames must match the abundance table colnames. Build the taxonomy module list with custom_modulelist(), supplying the genus-to-species mappings and genus-to-description mappings, replacing the built-in KEGG pathway/module databases. Compute per-species group statistics (p-values and log fold-changes per cutotype contrast) with ko.test(), then run reporter_score()/GRSA() with the custom taxonomy modulelist so species statistics are aggregated to genus-level reporter scores, with p-value adjustment at both the feature level (p.adjust.method1) and module level (p.adjust.method2). Do not prefilter the abundance table, since GRSA is threshold-free. Summarize enriched genera (e.g. plot_report()) and export the result table (e.g. grsa_taxonomy_cutotype_results.csv).

## Related tools

- **ReporterScore** (R package providing custom_modulelist(), ko.test(), reporter_score()/GRSA(), and plot_report() for taxonomy-level enrichment via a custom genus-species module list) — https://github.com/Asa12138/ReporterScore
- **devtools** (Used to install the GitHub development versions of ReporterScore and pcutils)
- **pcutils** (Dependency installed from GitHub for ReporterScore visualization support) — github.com/Asa12138/pcutils

## Examples

```
modulelist <- custom_modulelist(genus2species, genus2description); ko_stat <- ko.test(species_abundance, group="Cutotype", metadata=metadata); reporter_res <- reporter_score(ko_stat, modulelist=modulelist)
```

## Evaluation signals

- The output table grsa_taxonomy_cutotype_results.csv exists and contains genus-level reporter scores with feature-level (p.adjust.method1) and module-level (p.adjust.method2) adjustments
- Enriched genera reproduce the paper's pattern: Psychrobacter, Paracoccus, Chryseobacterium, Elizabethkingia, Deinococcus, Microbacterium enriched in the M-cutotype; Acidipropionibacterium, Staphylococcus, Corynebacterium, Cutibacterium enriched in the C-cutotype
- Metadata rownames match abundance table colnames (all(rownames(metadata) %in% colnames(abundance)) is TRUE or intersection > 0)
- Every genus in the module list maps to at least one species present in the abundance table, so reporter scores are computed for all genera

## Limitations

- ReporterScore package version number is not stated in the paper, limiting reproducibility
- GRSA exhibits slightly lower specificity than alternatives, so enriched genera require additional experimental validation
- Quantitative thresholds are not defined; validation against the paper's cutotype pattern is qualitative
- No established gold standard exists for comparing enrichment analysis methods, and true negatives are hard to define due to incomplete databases

## Evidence

- [readme] GRSA is a threshold-free method that works well with all types of biomedical features, such as genes, chemical compounds, and microbial species: "'GRSA' is a threshold-free method that works well with all types of biomedical features, such as genes, chemical compounds, and microbial species."
- [readme] customized databases via custom_modulelist: [evidence span withheld — anchor status: external_doc]
- [readme] abundance table should not be prefiltered: [evidence span withheld — anchor status: not_found]
- [methods] reporter_score signature with modulelist and two-stage p-adjustment: "`reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)`"
- [abstract] demonstrated application of GRSA beyond functional enrichment using a taxonomy database: "We applied GRSA to microbiome, transcriptome and metabolome data and discovered new biological insights in omics studies. Finally, we demonstrated the application of GRSA beyond functional enrichment"
