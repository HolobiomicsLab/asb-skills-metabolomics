---
name: kegg-api-orthology-retrieval
description: Use when a metabolomics data frame of KEGG compound identifiers contains
  significant metabolites (e.g. adjusted p <= 0.05) and you need the enzyme orthologies
  and genes an organism has for metabolizing those compounds.
when_to_use_negative:
- The input data frame lacks KEGG compound identifiers or was not loaded with read.metabo,
  so the KEGG column does not have the proper class and keggGet calls will fail
- The goal is compound hierarchy/class assignment rather than enzyme and gene orthology
  retrieval — assign_hierarchy alone on compound classes is the right choice
- No internet access to the KEGG API, since KEGG_gather retrieves metadata live via
  keggGet from KEGGREST
edam_operation: http://edamontology.org/operation_3209
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: omu
  role: R package providing KEGG_gather to fetch enzyme orthology and gene data per
    metabolite from the KEGG database, and assign_hierarchy to attach enzyme/organism
    metadata using the KO_Number identifier
  repo: github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: Supplies the keggGet function that KEGG_gather calls to query the KEGG API
    for orthology and gene metadata
- name: read.metabo
  role: Loads the metabolite statistics data frame into R so the KEGG column has the
    proper class before KEGG_gather is run
  repo: github.com/connor-reid-tiffany/Omu
- name: assign_hierarchy
  role: Attaches enzyme metadata and organism metadata to the KEGG_gather output using
    the KO_Number identifier and organism identifiers
  repo: github.com/connor-reid-tiffany/Omu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:533f482350dd256d3677651f75eb4b2d2e5080f0e130a8cef04eaac7252b54d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - KEGG_gather
  - R
  - assign_hierarchy
  - omu
  - read.metabo
  edam_operation: http://edamontology.org/operation_3209
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kegg-api-orthology-retrieval

## Summary

A method for retrieving all known enzyme orthology (KO) and gene annotations for a set of KEGG metabolite identifiers from the KEGG API via KEGGREST, using the omu R package's KEGG_gather and assign_hierarchy functions to link metabolites of interest to enzyme and gene data for organisms of interest.

## When to use

Use when a metabolomics data frame of KEGG compound identifiers contains significant metabolites (e.g. adjusted p <= 0.05) and you need the enzyme orthologies and genes an organism has for metabolizing those compounds.

## When NOT to use

- The input data frame lacks KEGG compound identifiers or was not loaded with read.metabo, so the KEGG column does not have the proper class and keggGet calls will fail
- The goal is compound hierarchy/class assignment rather than enzyme and gene orthology retrieval — assign_hierarchy alone on compound classes is the right choice
- No internet access to the KEGG API, since KEGG_gather retrieves metadata live via keggGet from KEGGREST

## Inputs

- Metabolomics statistics data frame with a KEGG compound identifier column, loaded via read.metabo so the KEGG column has the proper class
- Subset of metabolites of interest (e.g. compounds with adjusted p values <= 0.05 from a BH-adjusted student's t test or ANOVA in omu_summary/omu_anova)
- Organism identifiers (e.g. KEGG organism codes for Pseudomonas spp.) for gene-level annotation

## Outputs

- Data frame with appended functional orthology and KO_numbers columns from the KEGG API
- Combined orthology/gene annotation table saved as kegg_orthology_gene_annotations.csv, with enzyme metadata and organism metadata attached via assign_hierarchy

## How to apply

Load the metabolite statistics data frame with omu's read.metabo so the KEGG column has the proper class, then subset it to metabolites of interest (e.g. compounds with adjusted p values <= 0.05), since a compound can carry multiple functional orthologies. Run KEGG_gather on the subset: it calls keggGet from KEGGREST against the KEGG API and appends functional orthology and KO_numbers columns to the data frame. Run KEGG_gather again on the retrieved identifiers to gather orthology metadata and gene metadata from the KEGG API. Finally apply assign_hierarchy with the 'KO_Number' identifier (and organism identifiers for genes) to attach enzyme metadata and organism metadata, and save the combined table as kegg_orthology_gene_annotations.csv.

## Related tools

- **omu** (R package providing KEGG_gather to fetch enzyme orthology and gene data per metabolite from the KEGG database, and assign_hierarchy to attach enzyme/organism metadata using the KO_Number identifier) — github.com/connor-reid-tiffany/Omu
- **KEGGREST** (Supplies the keggGet function that KEGG_gather calls to query the KEGG API for orthology and gene metadata)
- **read.metabo** (Loads the metabolite statistics data frame into R so the KEGG column has the proper class before KEGG_gather is run) — github.com/connor-reid-tiffany/Omu
- **assign_hierarchy** (Attaches enzyme metadata and organism metadata to the KEGG_gather output using the KO_Number identifier and organism identifiers) — github.com/connor-reid-tiffany/Omu

## Examples

```
metabolites_sig <- subset(countDF, adj_pval <= 0.05); orth <- KEGG_gather(count_data = metabolites_sig, Group = "Metabolites"); orth_ko <- KEGG_gather(count_data = orth, Group = "KO_Number")
```

## Evaluation signals

- The output data frame contains the expected functional orthologies and KO_numbers columns appended after KEGG_gather, as the methods text states it should
- Every row in the retrieved table corresponds to a KEGG compound identifier from the pre-specified metabolite subset (e.g. adjusted p <= 0.05), not the full unfiltered count data
- assign_hierarchy on the KO_Number identifier successfully attaches enzyme metadata and organism metadata for the specified organisms of interest (e.g. Pseudomonas spp.)
- The final table is written to kegg_orthology_gene_annotations.csv with combined orthology and gene annotation columns
- Compounds with multiple associated functional orthologies are represented as multiple rows/entries rather than collapsed, matching the KEGG orthology mapping semantics

## Limitations

- Compounds can have multiple functional orthologies associated with them, so retrieving orthology on the full unfiltered data frame inflates the result; subsetting to metabolites of interest is required
- KEGG_gather depends on live calls to the KEGG API via keggGet from KEGGREST, so results depend on KEGG availability and current database content
- Compounds without KEGG numbers (unknowns) cannot be mapped to orthology/gene annotations; keep_unknowns handling only applies at the hierarchy-assignment stage, not to KEGG API retrieval

## Evidence

- [abstract] The KEGG_gather function can get all known enzyme orthology data and gene data associated with each metabolite from the KEGG database: "The KEGG_gather function can get all known enzyme orthology data and gene data associated with each metabolite from the KEGG database"
- [methods] keggGet from KEGGREST: "retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST"
- [methods] it is recommended to use the read.metabo function to load it into R: "it is recommended to use the ```read.metabo``` function to load it into R"
- [methods] To assign hierarchical class data, use the assign_hierarchy function: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
- [methods] a p value adjustment method of Benjamini & Hochberg with the argument p_adjust = "BH": "a p value adjustment method of Benjamini & Hochberg with the argument ```p_adjust = "BH"```"
