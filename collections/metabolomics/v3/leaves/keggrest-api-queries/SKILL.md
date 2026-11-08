---
name: keggrest-api-queries
description: Use when a metabolomics workflow yields compound IDs that must be linked
  to KEGG pathways and visualized on interactive pathway maps — e.g., the pathway
  enrichment step of MeTEor after metabolite ID conversion.
when_to_use_negative:
- Input metabolites are not convertible to KEGG compound IDs (e.g., no MetaboAnalyst
  ID conversion possible), so no KEGG pathway mapping can succeed
- The goal is unsupervised dimensionality reduction or classification of longitudinal
  metabolomics data (use MeTEor's PCA/TCAM or LR/RF/XGBoost modules instead)
- Pathway analysis must be done against databases other than KEGG (e.g., Reactome
  or MetaCyc), which KEGGREST cannot query
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: KEGGREST
  role: R/Bioconductor client used to map converted metabolite IDs to KEGG pathways
- name: MetaboAnalyst
  role: Web service used to convert raw metabolite identifiers into KEGG compound
    IDs before pathway mapping
- name: MeTEor
  role: R Shiny app whose pathway enrichment module orchestrates ID conversion, KEGGREST
    mapping, ranking, and interactive outlink generation
  repo: https://github.com/scibiome/meteor
- name: KEGG
  role: Reference database supplying pathway membership and pathway-map images for
    highlighting metabolites
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:7524bb9791ba404a2d071542637a18c766e7137fff1f4072691e197ac0b28d75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - KEGG
  - KEGGREST
  - MeTEor
  - MetaboAnalyst
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# keggrest-api-queries

## Summary

Query KEGG via KEGGREST to map converted metabolite IDs (HMDB/KEGG compound IDs obtained through MetaboAnalyst ID Conversion) to KEGG pathways, rank pathways by metabolite occurrence, and generate interactive KEGG pathway-map URLs with the identified metabolites highlighted.

## When to use

Use when a metabolomics workflow yields compound IDs that must be linked to KEGG pathways and visualized on interactive pathway maps — e.g., the pathway enrichment step of MeTEor after metabolite ID conversion.

## When NOT to use

- Input metabolites are not convertible to KEGG compound IDs (e.g., no MetaboAnalyst ID conversion possible), so no KEGG pathway mapping can succeed
- The goal is unsupervised dimensionality reduction or classification of longitudinal metabolomics data (use MeTEor's PCA/TCAM or LR/RF/XGBoost modules instead)
- Pathway analysis must be done against databases other than KEGG (e.g., Reactome or MetaCyc), which KEGGREST cannot query

## Inputs

- CSV of metabolite identifiers converted to KEGG compound IDs (via MetaboAnalyst ID Conversion)
- Metabolite IDs from MeTEor's pathway enrichment step 2 field

## Outputs

- Ranked pathway list ordered by descending occurrence count
- Interactive KEGG pathway-map URLs with identified metabolites highlighted in red
- enriched_pathways.csv containing pathway names, occurrence counts, and KEGG outlinks

## How to apply

First convert raw metabolite identifiers to KEGG compound IDs (in MeTEor, via MetaboAnalyst ID Conversion; note the article states the MetaboAnalyst API was unavailable, so IDs are pasted back as CSV content into the pathway enrichment step 2 field). Then use KEGGREST to map each converted metabolite ID to the KEGG pathways in which it occurs. Rank the resulting pathways by the number (occurrence) of query metabolites present in each pathway — MeTEor surfaces this via the 'Get enriched pathways' action. For each enriched pathway, construct a modified URL to the interactive KEGG pathway map such that the identified metabolites are highlighted in red. Finally, write the ranked pathway list (pathway names, occurrence counts, interactive KEGG outlinks) to enriched_pathways.csv.

## Related tools

- **KEGGREST** (R/Bioconductor client used to map converted metabolite IDs to KEGG pathways)
- **MetaboAnalyst** (Web service used to convert raw metabolite identifiers into KEGG compound IDs before pathway mapping)
- **MeTEor** (R Shiny app whose pathway enrichment module orchestrates ID conversion, KEGGREST mapping, ranking, and interactive outlink generation) — https://github.com/scibiome/meteor
- **KEGG** (Reference database supplying pathway membership and pathway-map images for highlighting metabolites)

## Examples

```
library(KEGGREST); pathways <- keggLink("pathway", cpd_ids); enriched_pathways.csv <- rank_by_occurrence(pathways)
```

## Evaluation signals

- Output file enriched_pathways.csv exists after the run
- Each row contains a pathway entry with an occurrence count and a KEGG pathway-map URL
- Rows are ordered by descending occurrence of the selected metabolites
- Each generated URL opens an interactive KEGG pathway map where the identified metabolites are highlighted in red

## Limitations

- MetaboAnalyst ID conversion depends on identifiers resolvable by MetaboAnalyst; the article notes 'The API of MetaboAnalyst is currently not available, therefore we developed this workaround' (CSV paste-back into step 2)
- KEGG IDs are required: 'The IDs are needed to map the metabolites to known KEGG pathways' — non-KEGG identifiers will not map without prior conversion
- Pathway ranking is by simple occurrence count of query metabolites, not by statistical enrichment p-value

## Evidence

- [methods] MeTEor queries metabolite IDs via MetaboAnalyst ID Conversion, then executes pathway analysis using KEGGREST: [evidence span withheld — anchor status: not_found]
- [methods] Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence.: "Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence."
- [methods] The IDs are needed to map the metabolites to known KEGG pathways.: "The IDs are needed to map the metabolites to known KEGG pathways."
- [readme] Map compounds (MetaboAnalyst ID Conversion), Map to KEGG Pathways (KEGGREST): [evidence span withheld — anchor status: external_doc]
- [methods] head to MetaboAnalyst for the ID conversion: "head to <a href='https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml' target='_blank'>MetaboAnalyst</a> for the ID conversion"
