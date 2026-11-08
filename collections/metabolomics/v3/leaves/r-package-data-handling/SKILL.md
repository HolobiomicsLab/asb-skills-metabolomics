---
name: r-package-data-handling
description: Use when a metabolite count data frame (e.g., c57_nos2KO_mouse_countDF)
  has KEGG compound numbers and you need per-metabolite compound class and subclass
  metadata attached before statistics or plotting.
when_to_use_negative:
- The metabolite identifiers are not KEGG compound numbers — assign_hierarchy maps
  hierarchy via KEGG identifiers, so other ID systems will not resolve to class metadata.
- You need statistical testing or fold-change output rather than metadata annotation
  — those are separate omu functions (omu_summary, omu_anova, count_fold_changes).
- You need gene/enzyme-level annotation for each metabolite — that is the role of
  KEGG_gather, not assign_hierarchy.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3489
tools:
- name: omu
  role: Provides the assign_hierarchy function and internal data tables that attach
    KEGG compound class/subclass metadata to each metabolite in a count data frame
  repo: github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: Underlying R package whose keggGet function retrieves compound data from the
    KEGG API during hierarchy assignment
- name: R
  role: Execution environment in which omu's assign_hierarchy is called on the count
    data frame
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:e0e42e289c36dff0a39bfa8023d5194157f93f75dbeb1c7fd2e9397b4d34bc40
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - R
  - omu
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# r-package-data-handling

## Summary

Annotate a metabolomics count data frame with KEGG compound hierarchy metadata (Class, Subclass_1–4) using the omu R package's assign_hierarchy function, which maps each KEGG compound number to hierarchical class metadata.

## When to use

Use when a metabolite count data frame (e.g., c57_nos2KO_mouse_countDF) has KEGG compound numbers and you need per-metabolite compound class and subclass metadata attached before statistics or plotting.

## When NOT to use

- The metabolite identifiers are not KEGG compound numbers — assign_hierarchy maps hierarchy via KEGG identifiers, so other ID systems will not resolve to class metadata.
- You need statistical testing or fold-change output rather than metadata annotation — those are separate omu functions (omu_summary, omu_anova, count_fold_changes).
- You need gene/enzyme-level annotation for each metabolite — that is the role of KEGG_gather, not assign_hierarchy.

## Inputs

- Metabolomics count data frame with KEGG compound identifiers (e.g., the omu example dataset c57_nos2KO_mouse_countDF of fecal samples from c57B6J WT vs nos2KO mice under mock vs streptomycin treatment)
- Identifier type argument (identifier = "KEGG")
- keep_unknowns logical flag controlling retention of compounds lacking KEGG numbers

## Outputs

- Annotated count data frame with added hierarchy columns (Class, Subclass_1, Subclass_2, Subclass_3, Subclass_4), one row per input metabolite
- CSV export of the annotated data frame (e.g., assign_hierarchy_KEGG.csv)

## How to apply

Load the count data with omu — for the package's own example format, the read.metabo function is recommended for loading metabolomics count data into R. Call assign_hierarchy(count_data = ..., keep_unknowns = ..., identifier = "KEGG") on the count data frame; omu retrieves data from the KEGG API using the keggGet function from KEGGREST and uses internal data tables to map each KEGG compound number to class, subclass 1, subclass 2, subclass 3, and subclass 4. Set keep_unknowns = TRUE to retain compounds without KEGG numbers so row counts match the input. The rationale is that assign_hierarchy supplies the metadata for every metabolite that downstream omu functions (omu_summary, count_fold_changes, plot_volcano) group by. Write the returned annotated data frame to CSV (e.g., assign_hierarchy_KEGG.csv) for validation and reuse.

## Related tools

- **omu** (Provides the assign_hierarchy function and internal data tables that attach KEGG compound class/subclass metadata to each metabolite in a count data frame) — github.com/connor-reid-tiffany/Omu
- **KEGGREST** (Underlying R package whose keggGet function retrieves compound data from the KEGG API during hierarchy assignment)
- **R** (Execution environment in which omu's assign_hierarchy is called on the count data frame)

## Examples

```
assign_hierarchy(count_data = c57_nos2KO_mouse_countDF, keep_unknowns = TRUE, identifier = "KEGG")
```

## Evaluation signals

- The output file (e.g., assign_hierarchy_KEGG.csv) exists and is readable.
- Row count of the annotated data frame equals the row count of the input count data frame (one row per metabolite), including retained unknowns when keep_unknowns = TRUE.
- The output contains the hierarchy metadata columns named as in the vignette's kable output of DF (e.g., Class, Subclass_1, Subclass_2).
- Every metabolite with a KEGG compound number carries a populated compound class entry; only compounds without KEGG numbers are unknown when keep_unknowns = TRUE.
- Hierarchy values follow KEGG's compound-class structure (Class, Subclass_1–4), so downstream omu_summary and count_fold_changes can group metabolites by these columns without schema errors.

## Limitations

- Compounds without KEGG compound numbers cannot be assigned hierarchy metadata; they are only kept in the output if keep_unknowns = TRUE.
- Metadata retrieval depends on the KEGG database (accessed via KEGGREST keggGet), so annotation completeness tracks KEGG's coverage of the compounds.
- Hierarchy assignment is limited to KEGG compound classes and subclasses; other classification systems are not supported by assign_hierarchy.

## Evidence

- [methods] To assign hierarchical class data, use the ```assign_hierarchy``` function: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
- [methods] keep_unknowns keeps compounds without KEGG numbers: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
- [abstract] omu is an R package designed for assigning compound hierarchies: "we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations for organisms of interest."
- [methods] Included with Omu is an example metabolomics dataset of fecal samples: "Included with Omu is an example metabolomics dataset of data from fecal samples collected from a two factor experiment"
- [abstract] omu provides graphing and statistical functions centered on assign_hierarchy: "omu provides a suite of graphing and statistical functions centered around the use of assign_hierarchy, which provides the metadata for each metabolite based on a KEGG identifier"
