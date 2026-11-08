---
name: kegg-compound-hierarchy-annotation
description: Use when a metabolomics count data frame has KEGG compound identifiers
  and downstream statistics, bar plots, or volcano plots need per-metabolite class/subclass
  metadata from the KEGG hierarchy.
when_to_use_negative:
- Metabolites lack KEGG compound numbers and only unknowns would be annotated — with
  keep_unknowns = FALSE these compounds would be dropped, so hierarchy annotation
  is uninformative.
- The goal is enzyme orthology or gene annotation rather than compound class metadata
  — KEGG_gather is the appropriate omu function, not assign_hierarchy.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
tools:
- name: omu
  role: R package providing the assign_hierarchy function that attaches KEGG compound
    class/subclass metadata to metabolites using internal system data tables
  repo: github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: supplies the keggGet function omu uses to retrieve hierarchical compound data
    from the KEGG API
- name: R
  role: runtime environment in which omu is executed to annotate the count data frame
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:2dc736a297e56422b7053df639245440139092eafcd0dd5b2e1b515017e9a0aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - R
  - assign_hierarchy
  - ggplot2
  - omu
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# kegg-compound-hierarchy-annotation

## Summary

Annotates each metabolite in an omu count data frame with hierarchical KEGG compound class metadata (Class, Subclass_1 through Subclass_4) using the omu assign_hierarchy function, which serves as the metadata backbone for omu_summary statistics, count_fold_changes, and plot functions.

## When to use

Use when a metabolomics count data frame has KEGG compound identifiers and downstream statistics, bar plots, or volcano plots need per-metabolite class/subclass metadata from the KEGG hierarchy.

## When NOT to use

- Metabolites lack KEGG compound numbers and only unknowns would be annotated — with keep_unknowns = FALSE these compounds would be dropped, so hierarchy annotation is uninformative.
- The goal is enzyme orthology or gene annotation rather than compound class metadata — KEGG_gather is the appropriate omu function, not assign_hierarchy.

## Inputs

- omu metabolomics count data frame with one row per metabolite and a KEGG compound identifier column (e.g. c57_nos2KO_mouse_countDF)
- identifier argument specifying the annotation type, here "KEGG"

## Outputs

- annotated data frame with one row per input metabolite and added KEGG hierarchy columns (e.g. Class, Subclass_1, Subclass_2, Subclass_3, Subclass_4), often written as assign_hierarchy_KEGG.csv

## How to apply

Load the metabolomics count data frame (e.g. the packaged c57_nos2KO_mouse_countDF of fecal samples from c57B6J wild type vs nos2 knockout mice under mock vs streptomycin treatment, loaded via read.metabo if external) and call assign_hierarchy with identifier = "KEGG". omu maps each KEGG compound number to its hierarchical metadata using internal system data tables, which retrieve data from the KEGG API via the keggGet function of the KEGGREST package. Set keep_unknowns = TRUE to retain compounds without KEGG numbers, which otherwise would be dropped; compounds lacking numbers are labeled NA in the hierarchy columns. The returned data frame has the same rows as the input plus added hierarchy columns (Class, Subclass_1, Subclass_2, etc.). This annotated frame is the required metadata input for count_fold_changes (column = "Class") and plot_volcano highlighting by compound class, as in Fig. 1A/1B of the paper.

## Related tools

- **omu** (R package providing the assign_hierarchy function that attaches KEGG compound class/subclass metadata to metabolites using internal system data tables) — github.com/connor-reid-tiffany/Omu
- **KEGGREST** (supplies the keggGet function omu uses to retrieve hierarchical compound data from the KEGG API)
- **R** (runtime environment in which omu is executed to annotate the count data frame)

## Examples

```
DF <- assign_hierarchy(count_data = c57_nos2KO_mouse_countDF, keep_unknowns = TRUE, identifier = "KEGG")
```

## Evaluation signals

- Output file (e.g. assign_hierarchy_KEGG.csv) exists and contains exactly one row per metabolite from the input count data frame.
- Annotated data frame includes the hierarchy metadata columns shown in the vignette's kable output of DF (e.g. Class, Subclass_1, Subclass_2).
- With keep_unknowns = TRUE, compounds without KEGG numbers are retained in the output rather than dropped.
- Downstream compatibility check: count_fold_changes(column = "Class", sig_threshold = 0.05) and plot_volcano(column = "Class") run on the annotated frame joined with omu_summary statistics without key errors.
- Every metabolite carrying a KEGG compound number receives non-NA Class/Subclass values (assuming the identifier maps in the KEGG tables).

## Limitations

- Annotation depends on the KEGG API via KEGGREST; compounds without KEGG compound numbers get no hierarchy and are only retained if keep_unknowns = TRUE.
- assign_hierarchy provides metadata only; statistical testing, fold change, and visualization require omu_summary, count_fold_changes, and plot functions downstream.
- Non-KEGG identifier types require the appropriate identifier argument; the documented example workflow uses identifier = "KEGG".

## Evidence

- [abstract] omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations: "we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations for organisms of interest."
- [methods] assign_hierarchy assigns hierarchical class data: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
- [methods] keep_unknowns retains compounds without KEGG numbers: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
- [methods] KEGGREST keggGet retrieval from KEGG API: "retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST"
- [methods] packaged example metabolomics dataset: "Included with Omu is an example metabolomics dataset of data from fecal samples collected from a two factor experiment"
