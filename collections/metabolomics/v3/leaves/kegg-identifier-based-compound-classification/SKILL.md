---
name: kegg-identifier-based-compound-classification
description: Use when a metabolomics count data frame contains KEGG compound identifiers
  and each feature needs hierarchical class/subclass metadata attached before statistical
  testing or figure faceting.
when_to_use_negative:
- Compounds lack KEGG compound numbers and keep_unknowns = FALSE is intended — those
  features cannot receive hierarchy metadata and will be dropped
- The goal is enzyme orthology or gene annotation lookup per metabolite rather than
  compound class hierarchy — use omu's KEGG_gather instead
- Input is raw instrument output (e.g. vendor peak lists) rather than a count table
  already keyed by compound identifier — load via read.metabo first
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: omu
  role: Provides assign_hierarchy and internal KEGG hierarchy data tables to attach
    compound class/subclass metadata to each metabolite
  repo: github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: Populates omu's hierarchy tables by retrieving data from the KEGG API via
    the keggGet function
- name: R
  role: Language environment in which omu runs and returns the annotated data frame
- name: read.metabo
  role: Recommended omu function to load the metabolomics count file into R before
    hierarchy assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:d06106ac9b9db38af4ace60737513218b36041ea564ddbc4353a2701324c1f7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - R
  - omu
  - read.metabo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# kegg-identifier-based-compound-classification

## Summary

Annotates each metabolite in a count data frame with hierarchical KEGG compound class metadata (Class, Subclass_1 through Subclass_4) using the omu R package's assign_hierarchy function, keyed on KEGG compound numbers.

## When to use

Use when a metabolomics count data frame contains KEGG compound identifiers and each feature needs hierarchical class/subclass metadata attached before statistical testing or figure faceting.

## When NOT to use

- Compounds lack KEGG compound numbers and keep_unknowns = FALSE is intended — those features cannot receive hierarchy metadata and will be dropped
- The goal is enzyme orthology or gene annotation lookup per metabolite rather than compound class hierarchy — use omu's KEGG_gather instead
- Input is raw instrument output (e.g. vendor peak lists) rather than a count table already keyed by compound identifier — load via read.metabo first

## Inputs

- Metabolomics count data frame with KEGG compound identifiers (e.g. the omu example dataset c57_nos2KO_mouse_countDF from fecal samples of c57B6J wild type vs nos2 knockout mice under mock vs streptomycin treatment)
- Optional metadata file (e.g. c57_nos2KO_mouse_metadata)

## Outputs

- Annotated count data frame with added hierarchy columns: Class, Subclass_1, Subclass_2, Subclass_3, Subclass_4 (e.g. written to assign_hierarchy_KEGG.csv)

## How to apply

Load the metabolomics count table into R (the omu-recommended path is the read.metabo function) so each row is a metabolite with a KEGG compound number column. Call assign_hierarchy with identifier = "KEGG"; omu maps each KEGG compound number to its hierarchical metadata via internal system data tables populated through keggGet from the KEGGREST package against the KEGG API. Set keep_unknowns = TRUE to retain compounds lacking KEGG numbers in the output rather than dropping them. The function returns the same count data frame with added hierarchy columns organized as compound class, subclass 1, subclass 2, subclass 3, and subclass 4; this annotated frame is the metadata backbone for downstream omu_summary t-tests, count_fold_changes category counts, and ggplot2-based figures. Validate that one row per input metabolite is returned and that hierarchy columns (e.g. Class, Subclass_1, Subclass_2) are populated, as shown in the package vignette's kable output of DF.

## Related tools

- **omu** (Provides assign_hierarchy and internal KEGG hierarchy data tables to attach compound class/subclass metadata to each metabolite) — github.com/connor-reid-tiffany/Omu
- **KEGGREST** (Populates omu's hierarchy tables by retrieving data from the KEGG API via the keggGet function)
- **R** (Language environment in which omu runs and returns the annotated data frame)
- **read.metabo** (Recommended omu function to load the metabolomics count file into R before hierarchy assignment)

## Examples

```
assign_hierarchy(count_data = c57_nos2KO_mouse_countDF, keep_unknowns = TRUE, identifier = "KEGG")
```

## Evaluation signals

- The returned data frame has exactly one row per metabolite from the input count data frame (no rows added or dropped when keep_unknowns = TRUE)
- Hierarchy metadata columns (Class, Subclass_1, Subclass_2, etc.) are present in the output and populated for metabolites with KEGG compound numbers
- Output written to assign_hierarchy_KEGG.csv exists and matches the annotated frame, mirroring the vignette's kable output of DF
- Compounds without KEGG numbers are retained in the output when keep_unknowns = TRUE
- Downstream omu_summary or count_fold_changes can group metabolites by the assigned KEGG compound category without key mismatches

## Limitations

- Hierarchy assignment depends on each metabolite having a KEGG compound number; compounds lacking identifiers carry no class metadata unless keep_unknowns = TRUE keeps them as unannotated rows
- Class assignments reflect the KEGG compound hierarchy as retrieved via KEGGREST from the KEGG API, so annotations are bounded by KEGG's coverage and current database content

## Evidence

- [methods] To assign hierarchical class data, use the ```assign_hierarchy``` function: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
- [abstract] we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations for organisms of interest: "we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations"
- [methods] ```keep_unknowns = TRUE``` keeps compounds without KEGG numbers: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
- [methods] retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST: "retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST"
- [abstract] omu provides a suite of graphing and statistical functions centered around the use of assign_hierarchy, which provides the metadata for each metabolite based on a KEGG identifier: "omu provides a suite of graphing and statistical functions centered around the use of assign_hierarchy, which provides the metadata for each metabolite based on a KEGG identifier"
