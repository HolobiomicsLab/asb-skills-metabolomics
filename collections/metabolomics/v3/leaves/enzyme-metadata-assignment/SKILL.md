---
name: enzyme-metadata-assignment
description: Use when a metabolomics data frame already carries KEGG compound IDs
  and you need the enzyme orthologies (KO numbers) and genes an organism of interest,
  e.g. Pseudomonas spp., has to metabolize a compound class such as organic acids.
when_to_use_negative:
- Compounds lack KEGG identifiers — KEGG_gather queries the KEGG API by KEGG compound
  ID, so unmapped/unknown metabolites cannot be gathered (keep_unknowns = TRUE only
  retains them without KEGG numbers in hierarchy assignment)
- The goal is compound class/hierarchy metadata alone — assign_hierarchy with KEGG
  compound identifiers suffices; the two-pass KEGG_gather orthology/gene retrieval
  is unnecessary
- Offline or rate-limited environments — the workflow makes live keggGet calls to
  the KEGG API via KEGGREST and requires network access
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: omu
  role: R package providing KEGG_gather and assign_hierarchy to link metabolites to
    enzyme orthology and gene annotations from KEGG
  repo: https://github.com/connor-reid-tiffany/Omu
- name: KEGG_gather
  role: omu function that retrieves all known enzyme orthology data and gene data
    associated with each metabolite from the KEGG database; run twice (compounds,
    then identifiers)
  repo: https://github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: supplies keggGet, which KEGG_gather calls against the KEGG API to fetch orthology
    and gene metadata
- name: assign_hierarchy
  role: omu function applied with the 'KO_Number' identifier and organism identifiers
    to attach enzyme metadata and organism metadata to the gathered table
  repo: https://github.com/connor-reid-tiffany/Omu
- name: read.metabo
  role: omu function used to load the metabolite count data frame so the KEGG column
    has the proper class
  repo: https://github.com/connor-reid-tiffany/Omu
- name: R
  role: language in which the omu workflow is implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:f3a1bf54281807458d308a120433d52df8e700a851a07d3af883adbf8eecad86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/connor-reid-tiffany/Omu
  tools:
  - KEGGREST
  - KEGG_gather
  - R
  - assign_hierarchy
  - omu
  - read.metabo
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# enzyme-metadata-assignment

## Summary

A workflow from the omu R package that links metabolomics compounds (KEGG compound identifiers) to enzyme functional orthologies (KO numbers) and gene annotations for organisms of interest, retrieved live from the KEGG API via KEGGREST, to support hypothesis generation.

## When to use

Use when a metabolomics data frame already carries KEGG compound IDs and you need the enzyme orthologies (KO numbers) and genes an organism of interest, e.g. Pseudomonas spp., has to metabolize a compound class such as organic acids.

## When NOT to use

- Compounds lack KEGG identifiers — KEGG_gather queries the KEGG API by KEGG compound ID, so unmapped/unknown metabolites cannot be gathered (keep_unknowns = TRUE only retains them without KEGG numbers in hierarchy assignment)
- The goal is compound class/hierarchy metadata alone — assign_hierarchy with KEGG compound identifiers suffices; the two-pass KEGG_gather orthology/gene retrieval is unnecessary
- Offline or rate-limited environments — the workflow makes live keggGet calls to the KEGG API via KEGGREST and requires network access

## Inputs

- metabolite statistics data frame containing KEGG compound identifiers, loaded with read.metabo
- subset of metabolites of interest (e.g. adjusted p value <= 0.05)
- KEGG compound identifiers (e.g. cpd numbers)
- organism identifiers for gene lookup

## Outputs

- data frame with appended functional orthology and KO_numbers columns
- combined orthology/gene annotation table (e.g. kegg_orthology_gene_annotations.csv) with enzyme metadata and organism metadata attached via assign_hierarchy

## How to apply

Load the metabolite statistics data frame with omu's read.metabo so the KEGG column has the proper class, then subset to metabolites of interest (e.g. compounds with adjusted p values <= 0.05), since compounds can have multiple functional orthologies associated with them. Run omu's KEGG_gather on the subset; it calls keggGet from KEGGREST against the KEGG API and appends functional orthology and KO_numbers columns to the data frame. Run KEGG_gather again on the retrieved identifiers to gather orthology metadata and gene metadata from the KEGG API. Finally apply assign_hierarchy with the 'KO_Number' identifier (plus organism identifiers for genes) to attach enzyme and organism metadata, and save the combined table, e.g. as kegg_orthology_gene_annotations.csv.

## Related tools

- **omu** (R package providing KEGG_gather and assign_hierarchy to link metabolites to enzyme orthology and gene annotations from KEGG) — https://github.com/connor-reid-tiffany/Omu
- **KEGG_gather** (omu function that retrieves all known enzyme orthology data and gene data associated with each metabolite from the KEGG database; run twice (compounds, then identifiers)) — https://github.com/connor-reid-tiffany/Omu
- **KEGGREST** (supplies keggGet, which KEGG_gather calls against the KEGG API to fetch orthology and gene metadata)
- **assign_hierarchy** (omu function applied with the 'KO_Number' identifier and organism identifiers to attach enzyme metadata and organism metadata to the gathered table) — https://github.com/connor-reid-tiffany/Omu
- **read.metabo** (omu function used to load the metabolite count data frame so the KEGG column has the proper class) — https://github.com/connor-reid-tiffany/Omu
- **R** (language in which the omu workflow is implemented)

## Examples

```
countDF <- read.metabo(file = "metabolite_stats.csv"); kegg_df <- KEGG_gather(KEGG_data = subset(countDF, adjusted_pvalue <= 0.05), sig_threshold = 0.05); kegg_df <- KEGG_gather(KEGG_data = kegg_df, sig_threshold = 0.05); kegg_df <- assign_hierarchy(Count_df = kegg_df, keep_unknowns = TRUE, identifier = "KO_Number")
```

## Evaluation signals

- After the first KEGG_gather call, confirm the output data frame has functional orthologies and KO_numbers columns added, as the methods text states it should
- Each KEGG compound in the subset may map to multiple functional orthologies — verify one-to-many rows are retained rather than collapsed
- After assign_hierarchy with 'KO_Number', confirm enzyme metadata and organism metadata columns are populated for the organism of interest
- Check that all retrieved orthology and gene metadata trace back to live KEGG API responses (keggGet via KEGGREST), i.e. non-empty entries for compounds with known KEGG IDs
- The final combined table should serialize cleanly, e.g. as kegg_orthology_gene_annotations.csv, with compound, KO, and gene columns present

## Limitations

- Compounds without KEGG numbers cannot be linked to orthology/gene data; unknowns are only optionally kept (keep_unknowns = TRUE) during hierarchy assignment
- Because compounds can have multiple functional orthologies associated with them, the table expands — subsetting to metabolites of interest (e.g. adjusted p values <= 0.05) before gathering is advised to control size
- Two successive KEGG_gather calls hit the KEGG API, so results depend on KEGG database/API availability and current KEGG content

## Evidence

- [abstract] we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations for organisms of interest: "we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations for organisms of interest"
- [abstract] The KEGG_gather function can get all known enzyme orthology data and gene data associated with each metabolite from the KEGG database: "The KEGG_gather function can get all known enzyme orthology data and gene data associated with each metabolite from the KEGG database"
- [methods] retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST: "retrieves data from the KEGG API using the function ```keggGet``` from the package KEGGREST"
- [methods] it is recommended to use the ```read.metabo``` function to load it into R: "it is recommended to use the ```read.metabo``` function to load it into R"
- [methods] To assign hierarchical class data, use the ```assign_hierarchy``` function: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
