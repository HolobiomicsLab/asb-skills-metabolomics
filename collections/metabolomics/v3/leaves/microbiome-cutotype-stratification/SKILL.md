---
name: microbiome-cutotype-stratification
description: Use when you have a species-level microbial abundance table plus sample
  metadata with group labels (e.g., cutotypes) and want genus-level enrichment of
  taxa rather than KEGG pathway/functional enrichment.
when_to_use_negative:
- The input is a KO/gene/compound abundance table and the goal is standard KEGG pathway
  or module functional enrichment — use the built-in KOlist/CPDlist databases instead
  of a custom taxonomy modulelist
- The abundance table has already been prefiltered to a shortlist of significant taxa,
  since GRSA is a threshold-free method that needs the full background to aggregate
  statistics
- Only a single-group sample set exists with no metadata grouping — ko.test() requires
  at least two categories (or time points/continuous variables) for the differential
  statistics
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ReporterScore
  role: R package implementing GRSA; custom_modulelist() builds the genus-species
    module list, ko.test() computes species-level group statistics, reporter_score()/GRSA()
    aggregates to genus-level reporter scores, plot_report() visualizes enriched genera
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency of ReporterScore installed from GitHub before loading the package
  repo: github.com/Asa12138/pcutils
- name: devtools
  role: Used to install the GitHub development versions of pcutils and ReporterScore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:4d1515f45cd087cc9d2dbe58867a868d40c0f53c672dd6a4abbfd127e5f3f9db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# microbiome-cutotype-stratification

## Summary

Extends GRSA beyond functional enrichment to taxonomy-level enrichment by building a custom genus-to-species hierarchical module list and aggregating species-level differential-abundance statistics to genus-level reporter scores, as demonstrated on the IHSMGC skin microbiome cutotype dataset.

## When to use

Use when you have a species-level microbial abundance table plus sample metadata with group labels (e.g., cutotypes) and want genus-level enrichment of taxa rather than KEGG pathway/functional enrichment.

## When NOT to use

- The input is a KO/gene/compound abundance table and the goal is standard KEGG pathway or module functional enrichment — use the built-in KOlist/CPDlist databases instead of a custom taxonomy modulelist
- The abundance table has already been prefiltered to a shortlist of significant taxa, since GRSA is a threshold-free method that needs the full background to aggregate statistics
- Only a single-group sample set exists with no metadata grouping — ko.test() requires at least two categories (or time points/continuous variables) for the differential statistics

## Inputs

- Species-level microbial abundance table (rownames = species, colnames = samples; read counts or relative abundance)
- Sample metadata table (rownames = samples) with a categorical grouping variable such as cutotype
- Custom taxonomy module list built with custom_modulelist() mapping genera to member species, plus genus descriptions

## Outputs

- Genus-level enrichment result table (reporter scores with feature- and module-level adjusted p-values), e.g. grsa_taxonomy_cutotype_results.csv
- Visualization of enriched genera per group contrast (e.g., via plot_report())

## How to apply

Load the species-level abundance table and metadata into R, ensuring metadata rownames match abundance table colnames. Build a taxonomy module list mapping each genus to its member species with custom_modulelist(), supplying genus-to-species and genus-to-description mappings. Compute per-species group statistics (p-values and log fold-changes for each cutotype contrast) with ko.test(), which handles the metadata grouping. Run reporter_score()/GRSA() with the custom taxonomy modulelist so species-level statistics are aggregated to genus-level reporter scores, with p-value adjustment applied at both the feature level (p.adjust.method1) and the module level (p.adjust.method2); do not prefilter the abundance table since GRSA is threshold-free. Summarize and visualize enriched genera with plot_report() and export the enrichment table. On the IHSMGC dataset this recovered M-cutotype-enriched Psychrobacter, Paracoccus, Chryseobacterium, Elizabethkingia, Deinococcus, Microbacterium versus C-cutotype-enriched Acidipropionibacterium, Staphylococcus, Corynebacterium, Cutibacterium.

## Related tools

- **ReporterScore** (R package implementing GRSA; custom_modulelist() builds the genus-species module list, ko.test() computes species-level group statistics, reporter_score()/GRSA() aggregates to genus-level reporter scores, plot_report() visualizes enriched genera) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency of ReporterScore installed from GitHub before loading the package) — github.com/Asa12138/pcutils
- **devtools** (Used to install the GitHub development versions of pcutils and ReporterScore)

## Examples

```
reporter_score(kodf = species_abundance, group = "Cutotype", metadata = metadata, type = "custom", feature = "species", modulelist = custom_modulelist(genus_species, genus_desc))
```

## Evaluation signals

- The output enrichment table (e.g., grsa_taxonomy_cutotype_results.csv) exists and contains genus-level reporter scores with adjusted p-values
- Reporter-score-enriched genera differentiate the M- and C-cutotypes from the other cutotypes, qualitatively reproducing the paper's pattern (Psychrobacter/Paracoccus/Chryseobacterium up in M; Staphylococcus/Corynebacterium/Cutibacterium up in C)
- Metadata rownames match abundance table colnames (all(rownames(metadata) %in% colnames(abundance)) or at least non-empty intersect)
- The custom modulelist maps every species in the abundance table to exactly one genus, so aggregation covers the features rather than dropping them
- No prefiltering was applied to the abundance table before ko.test(), consistent with the threshold-free GRSA design

## Limitations

- GRSA exhibits slightly lower specificity, so enrichment results require additional experimental validation
- There is no established gold standard for comparing enrichment methods and specificity is hard to evaluate because true negatives are difficult to define due to database incompleteness
- Correctness depends on the quality of the custom genus-to-species mapping supplied to custom_modulelist(); no built-in taxonomy database substitutes for it

## Evidence

- [readme] GRSA' is a threshold-free method that works well with all types of biomedical features, such as genes, chemical compounds, and microbial species.: "‘GRSA’ is a threshold-free method that works well with all types of biomedical features, such as genes, chemical compounds, and microbial species."
- [readme] You can just customize your own pathway databases (gene set of interest) by using custom_modulelist().: [evidence span withheld — anchor status: external_doc]
- [other] Using the custom_modulelist function on the IHSMGC species abundance table, GRSA found Psychrobacter, Paracoccus, Chryseobacterium, Elizabethkingia, Deinococcus and Microbacterium enriched in the M-cutotype.: [evidence span withheld — anchor status: not_found]
- [other] Construct a taxonomy module list mapping each genus to its member species with custom_modulelist(), supplying the genus-to-species and genus-to-description mappings.: [evidence span withheld — anchor status: not_found]
- [readme] the input abundance table should not be prefiltered to retain the background information, as the ‘GRSA’ is a threshold-free method.: [evidence span withheld — anchor status: external_doc]
