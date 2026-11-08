---
name: metabolite-metadata-assignment
description: Use when a metabolomics count data frame (KEGG compound IDs) needs hierarchical
  compound-class annotation before differential analysis, volcano plots, or fold-change
  counting.
when_to_use_negative:
- Input data lacks KEGG compound identifiers, so no hierarchical metadata can be assigned
- The goal is already-complete downstream analysis (statistics, volcano plots) that
  omu_summary or plot_volcano handle directly
- Working with non-metabolomics data types (e.g. 16S sequence data) outside omu's
  scope
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3407
tools:
- name: omu
  role: R package providing assign_hierarchy and internal KEGG system tables for compound-class
    annotation
  repo: https://github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: Used via keggGet to retrieve KEGG compound hierarchy data from the KEGG API
- name: R
  role: Runtime environment in which omu and assign_hierarchy execute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:935fc624bb19a605053f9c43681b5559f4a552da37b2aacee5d571440b953a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/connor-reid-tiffany/Omu
  tools:
  - KEGGREST
  - R
  - omu
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolite-metadata-assignment

## Summary

Annotates metabolites in a count data frame with KEGG hierarchical compound-class metadata (compound class, subclass 1–4) using the omu R package's assign_hierarchy function, with an option to keep unknowns.

## When to use

Use when a metabolomics count data frame (KEGG compound IDs) needs hierarchical compound-class annotation before differential analysis, volcano plots, or fold-change counting.

## When NOT to use

- Input data lacks KEGG compound identifiers, so no hierarchical metadata can be assigned
- The goal is already-complete downstream analysis (statistics, volcano plots) that omu_summary or plot_volcano handle directly
- Working with non-metabolomics data types (e.g. 16S sequence data) outside omu's scope

## Inputs

- Metabolomics count data frame with KEGG compound identifiers (e.g. c57_nos2KO_mouse_countDF)

## Outputs

- Annotated count data frame with added hierarchy metadata columns (Class, Subclass_1, Subclass_2, etc.)

## How to apply

Load metabolomics count data (e.g. the omu example dataset c57_nos2KO_mouse_countDF from fecal samples of c57B6J wild type vs nos2 knockout mice under mock vs streptomycin treatment) via read.metabo, then call assign_hierarchy(count_data, keep_unknowns = TRUE, identifier = "KEGG") so each metabolite carrying a KEGG compound number receives hierarchical metadata from omu's internal system tables, which use keggGet from KEGGREST to draw on the KEGG API. The keep_unknowns argument controls whether compounds without KEGG numbers are retained. The function underpins downstream omu_summary statistical testing, count_fold_changes, plot_volcano, and KEGG_gather, and its output can feed ggplot2 figures. Validate by confirming one row per input metabolite and the presence of hierarchy columns (e.g. Class, Subclass_1, Subclass_2).

## Related tools

- **omu** (R package providing assign_hierarchy and internal KEGG system tables for compound-class annotation) — https://github.com/connor-reid-tiffany/Omu
- **KEGGREST** (Used via keggGet to retrieve KEGG compound hierarchy data from the KEGG API)
- **R** (Runtime environment in which omu and assign_hierarchy execute)

## Examples

```
assign_hierarchy(count_data = c57_nos2KO_mouse_countDF, keep_unknowns = TRUE, identifier = "KEGG")
```

## Evaluation signals

- Output data frame has one row per input metabolite
- Hierarchy columns (Class, Subclass_1, Subclass_2, etc.) present in the output
- Compounds without KEGG numbers retained when keep_unknowns = TRUE
- Identifiers remain unchanged so downstream omu_summary/count_fold_changes can join on them

## Limitations

- Annotation depends on identifiers being valid KEGG compound numbers; non-KEGG ID formats cannot be mapped by assign_hierarchy directly
- Compounds absent from KEGG receive no hierarchy unless keep_unknowns = TRUE, in which case they pass through with missing metadata
- KEGG compound-class depth (subclasses 1–4) is incomplete for some metabolites, so downstream counting per category may undercount

## Evidence

- [abstract] assigning hierarchical metadata from the Kyoto Encyclopedia of Genes and Genomes (KEGG) to each metabolite: [evidence span withheld — anchor status: not_found]
- [methods] To assign hierarchical class data, use the assign_hierarchy function: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
- [methods] keeps compounds without KEGG numbers: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
- [methods] retrieves data from the KEGG API using the function keggGet from the package KEGGREST: "retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST"
- [methods] Included with Omu is an example metabolomics dataset of data from fecal samples collected from a two factor experiment: "Included with Omu is an example metabolomics dataset of data from fecal samples collected from a two factor experiment"
