---
name: reporter-score-taxonomic-enrichment-analysis
description: Use when you have a species-level microbial abundance table plus sample
  metadata and want threshold-free, hierarchical enrichment of taxa (e.g., genera)
  across group contrasts instead of pathway/KO enrichment.
when_to_use_negative:
- Input is already a KO/gene/compound abundance table — use the built-in KEGG pathway,
  module, or GO databases (load_KOlist, load_GOlist, custom_modulelist_from_org) instead
  of a custom taxonomy module list
- You need differential abundance of individual species rather than hierarchical enrichment
  of higher taxa — ko.test() per-species statistics alone suffice
- The abundance table has been prefiltered to significant features — GRSA is threshold-free
  and the README warns input tables should retain background information
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: ReporterScore
  role: Implements GRSA; custom_modulelist() builds the genus-species module list,
    ko.test() computes species-level statistics, reporter_score()/GRSA() aggregates
    to genus-level reporter scores, plot_report() visualizes
  repo: https://github.com/Asa12138/ReporterScore
- name: devtools
  role: Used to install the development versions of pcutils and ReporterScore from
    GitHub
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency installed before ReporterScore for the development installation
    path
  repo: https://github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:e631a2ce47d597368f77ca22e948d77b713b6a5cb44f3c6669287ad26a56b176
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reporter-score-taxonomic-enrichment-analysis

## Summary

Extends Generalized Reporter Score-based Analysis (GRSA) from functional to taxonomy-level enrichment by supplying a custom hierarchical genus-to-species module list to the ReporterScore package, aggregating species-level statistics into genus-level reporter scores.

## When to use

Use when you have a species-level microbial abundance table plus sample metadata and want threshold-free, hierarchical enrichment of taxa (e.g., genera) across group contrasts instead of pathway/KO enrichment.

## When NOT to use

- Input is already a KO/gene/compound abundance table — use the built-in KEGG pathway, module, or GO databases (load_KOlist, load_GOlist, custom_modulelist_from_org) instead of a custom taxonomy module list
- You need differential abundance of individual species rather than hierarchical enrichment of higher taxa — ko.test() per-species statistics alone suffice
- The abundance table has been prefiltered to significant features — GRSA is threshold-free and the README warns input tables should retain background information

## Inputs

- Species-level microbial abundance table (rownames = species, colnames = samples, not prefiltered)
- Sample metadata table (rownames = samples matching abundance table colnames, grouping column e.g. cutotype)
- Custom genus-to-species mapping with genus descriptions (module list for custom_modulelist())

## Outputs

- Genus-level reporter score enrichment table with adjusted p-values per group contrast (e.g., grsa_taxonomy_cutotype_results.csv)
- Visualization of enriched genera (plot_report() output)

## How to apply

Load the species abundance table and metadata (e.g., the IHSMGC skin microbiome cutotype groups) in R and install ReporterScore via devtools::install_github("Asa12138/ReporterScore") after pcutils. Build a taxonomy module list mapping each genus to its member species and descriptions with custom_modulelist(), mirroring the packaged KO_abundance example structure. Compute per-species group statistics (p-values and log fold-changes per cutotype contrast) with ko.test(), applying p-value adjustment at the feature level (p.adjust.method1). Run reporter_score()/GRSA() with the custom taxonomy modulelist so species-level statistics are aggregated to genus-level reporter scores, with module-level adjustment (p.adjust.method2) applied; the method is threshold-free so the abundance table should NOT be prefiltered. Summarize and visualize enriched genera with plot_report() and export the enrichment table (e.g., grsa_taxonomy_cutotype_results.csv).

## Related tools

- **ReporterScore** (Implements GRSA; custom_modulelist() builds the genus-species module list, ko.test() computes species-level statistics, reporter_score()/GRSA() aggregates to genus-level reporter scores, plot_report() visualizes) — https://github.com/Asa12138/ReporterScore
- **devtools** (Used to install the development versions of pcutils and ReporterScore from GitHub) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency installed before ReporterScore for the development installation path) — https://github.com/Asa12138/pcutils

## Examples

```
modlist <- custom_modulelist(module2species, module2description); ko_stat <- ko.test(species_abundance, "Group", metadata); reporter_res <- reporter_score(ko_stat, group="Group", metadata=metadata, modulelist=modlist)
```

## Evaluation signals

- The enrichment result table (e.g., grsa_taxonomy_cutotype_results.csv) exists and contains genus-level reporter scores with module-level adjusted p-values
- Reporter-score-enriched genera qualitatively differentiate the M- and C-cutotypes, reproducing the paper's pattern (e.g., Psychrobacter, Paracoccus, Chryseobacterium, Elizabethkingia, Deinococcus, Microbacterium in M-cutotype; Acidipropionibacterium, Staphylococcus, Corynebacterium, Cutibacterium in C-cutotype)
- rownames of metadata match or partially match colnames of the abundance table (all(rownames(metadata) %in% colnames(abundance)) is TRUE)
- Every genus in the module list maps to at least one species present in the abundance table (min_exist_KO/max_exist_KO constraints satisfied)
- Feature-level (p.adjust.method1) and module-level (p.adjust.method2) p-value adjustments were both applied in the reporter_score() call

## Limitations

- GRSA exhibits a slightly lower specificity than alternative methods, underscoring the necessity for additional experimental validations
- It is difficult to define true negatives due to the complexity of biological systems, so specificity of enrichment cannot be accurately evaluated
- There is no established gold standard in the field for comparing enrichment analysis methods

## Evidence

- [other] Using the custom_modulelist function on the IHSMGC species abundance table, GRSA found Psychrobacter, Paracoccus, Chryseobacterium, Elizabethkingia, Deinococcus and Microbacterium enriched in the M-cutotype: [evidence span withheld — anchor status: not_found]
- [methods] custom modulelist function signature: "reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)"
- [readme] GRSA threshold-free description: "‘GRSA’ is a threshold-free method that works well with all types of biomedical features, such as genes, chemical compounds, and microbial species."
- [readme] input prefilter warning: [evidence span withheld — anchor status: external_doc]
- [abstract] beyond functional enrichment claim: "We applied GRSA to microbiome, transcriptome and metabolome data and discovered new biological insights in omics studies. Finally, we demonstrated the application of GRSA beyond functional enrichment"
