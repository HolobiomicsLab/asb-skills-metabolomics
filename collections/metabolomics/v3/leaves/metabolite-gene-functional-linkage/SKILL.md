---
name: metabolite-gene-functional-linkage
description: Use when metabolite statistics data frames with KEGG compound identifiers
  (e.g. metabolites with adjusted p values <= 0.05) need functional orthology and
  gene metadata for hypothesis generation.
when_to_use_negative:
- When the input data frame lacks KEGG compound identifiers or was not loaded with
  read.metabo, since the KEGG column class must be proper for KEGG_gather to query
  the KEGG API.
- When you only need hierarchical compound classification without enzyme/gene linkage
  — plain assign_hierarchy on compound identifiers is sufficient.
- When no subset to metabolites of interest has been made — running KEGG_gather on
  every compound is inappropriate because compounds can have multiple functional orthologies
  and KEGGREST calls would be excessive.
edam_operation: http://edamontology.org/operation_3793
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3174
tools:
- name: omu
  role: R package providing KEGG_gather, read.metabo and assign_hierarchy, which together
    assign compound hierarchies and link compounds to corresponding enzyme and gene
    annotations for organisms of interest
  repo: github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: Provides the keggGet function used by KEGG_gather to retrieve enzyme orthology
    and gene data from the KEGG API
- name: R
  role: Runtime environment in which omu and its KEGG_gather/assign_hierarchy functions
    execute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:ee80ca33479a35e5a5fd0d6b7f0a6cd38a4466868c0e1ba9653341b6b62fdbd2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - R
  - omu
  - read.metabo
  edam_operation: http://edamontology.org/operation_3793
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolite-gene-functional-linkage

## Summary

Links metabolomics compounds to their known enzyme orthologies and genes via KEGG_gather, which queries the KEGG API through KEGGREST, and annotates organisms and enzymes onto the resulting table with assign_hierarchy — supporting hypothesis generation about which organisms could metabolize compounds of interest.

## When to use

Use when metabolite statistics data frames with KEGG compound identifiers (e.g. metabolites with adjusted p values <= 0.05) need functional orthology and gene metadata for hypothesis generation.

## When NOT to use

- When the input data frame lacks KEGG compound identifiers or was not loaded with read.metabo, since the KEGG column class must be proper for KEGG_gather to query the KEGG API.
- When you only need hierarchical compound classification without enzyme/gene linkage — plain assign_hierarchy on compound identifiers is sufficient.
- When no subset to metabolites of interest has been made — running KEGG_gather on every compound is inappropriate because compounds can have multiple functional orthologies and KEGGREST calls would be excessive.

## Inputs

- Metabolite statistics data frame containing KEGG compound identifiers, loaded with omu's read.metabo so the KEGG column has the proper class
- Subset of metabolites of interest (e.g. compounds with adjusted p values <= 0.05)
- Organism identifiers for gene-level annotation

## Outputs

- Data frame with appended functional orthology and KO_numbers columns from KEGG_gather
- Table of enzyme and gene orthology metadata with assign_hierarchy annotations (e.g. kegg_orthology_gene_annotations.csv)

## How to apply

Load the metabolite statistics data frame with omu's read_metabo so the KEGG column has the proper class, then subset to metabolites of interest (e.g. compounds with adjusted p values <= 0.05) since a compound can have multiple functional orthologies. Run omu's KEGG_gather, which calls keggGet from KEGGREST against the KEGG API, appending functional orthology and KO_numbers columns to the data frame. Run KEGG_gather again on the retrieved identifiers to pull orthology and gene metadata from the KEGG API. Apply assign_hierarchy with the 'KO_Number' identifier, plus organism identifiers for genes, to attach enzyme and organism metadata. Save the combined orthology/gene annotation table (e.g. as kegg_orthology_gene_annotations.csv) and validate that the data frame gained functional orthologies and KO_numbers columns after KEGG_gather.

## Related tools

- **omu** (R package providing KEGG_gather, read.metabo and assign_hierarchy, which together assign compound hierarchies and link compounds to corresponding enzyme and gene annotations for organisms of interest) — github.com/connor-reid-tiffany/Omu
- **KEGGREST** (Provides the keggGet function used by KEGG_gather to retrieve enzyme orthology and gene data from the KEGG API)
- **R** (Runtime environment in which omu and its KEGG_gather/assign_hierarchy functions execute)

## Examples

```
kegg_orthology <- KEGG_gather(count_data = kegg_orthology, Gather = "functional_orthology") ; kegg_orthology <- assign_hierarchy(count_data = kegg_orthology, identifier = "KO_Number")
```

## Evaluation signals

- After the first KEGG_gather run, confirm the data frame has functional orthologies and KO_numbers columns added
- After the second KEGG_gather run, confirm orthology metadata and gene metadata from the KEGG API are present for each retrieved identifier
- After assign_hierarchy with the 'KO_Number' identifier, confirm enzyme and organism metadata columns are attached to the table
- Confirm the final table (e.g. kegg_orthology_gene_annotations.csv) covers only the metabolites of interest, not the full unfiltered compound list
- Confirm input was loaded with read.metabo so the KEGG column has the proper class before any API calls

## Limitations

- A compound can have multiple functional orthologies associated with it, so KEGG_gather should be applied to a subset of metabolites of interest rather than the whole data frame.
- Metabolites without KEGG numbers cannot be annotated unless keep_unknowns = TRUE is set.
- KEGG_gather depends on keggGet from KEGGREST reaching the KEGG API, so results rely on KEGG identifier coverage and API availability.

## Evidence

- [methods] KEGG_gather appends functional orthology and KO_numbers columns via KEGGREST: "retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST"
- [abstract] omu links metabolites to enzyme orthology and gene annotations: "The KEGG_gather function can get all known enzyme orthology data and gene data associated with each metabolite from the KEGG database"
- [abstract] omu's purpose includes enzyme/gene linkage for hypothesis generation: "omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations for organisms of interest"
- [abstract] assign_hierarchy attaches hierarchy metadata from KEGG: [evidence span withheld — anchor status: not_found]
