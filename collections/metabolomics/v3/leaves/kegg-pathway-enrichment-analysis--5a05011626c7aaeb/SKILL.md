---
name: kegg-pathway-enrichment-analysis
description: Use when a metabolite list (KEGG compound IDs converted via MetaboAnalyst
  ID Conversion) must be mapped to KEGG pathways to find where the selected metabolites
  occur and ranked by occurrence count.
when_to_use_negative:
- The input metabolites lack metabolite identifiers convertible to KEGG compound IDs,
  so MetaboAnalyst ID Conversion cannot produce the KEGG IDs KEGGREST requires
- The goal is pathway enrichment for gene/transcript identifiers (KEGGREST mapping
  here targets metabolite compound IDs, not gene IDs)
- A metabolite-level statistical test (e.g., Repeated Measures ANOVA, linear mixed
  models) is needed first — this skill operates on an already-selected metabolite
  list
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MeTEor
  role: R Shiny app whose Pathway Enrichment module orchestrates pathway mapping,
    ranking by occurrence, and export of enriched_pathways.csv
  repo: https://github.com/scibiome/meteor
- name: KEGGREST
  role: R client used to map converted metabolite IDs to the KEGG pathways in which
    they occur
- name: MetaboAnalyst
  role: External web service (ID Conversion at metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml)
    that converts metabolite identifiers into KEGG compound IDs
- name: KEGG
  role: Pathway database providing the pathway definitions and interactive pathway
    maps highlighted by modified URLs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:2283f98ae3667cbc719151578df8e0f29d594ba8a1ec5778caa7b8affc875896
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - KEGG
  - KEGGREST
  - MeTEor
  - MetaboAnalyst
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# kegg-pathway-enrichment-analysis

## Summary

Maps metabolites of interest, identified via MetaboAnalyst ID conversion, to KEGG pathways using KEGGREST in MeTEor and produces an occurrence-ranked pathway list with interactive KEGG pathway-map URLs that highlight the selected metabolites in red.

## When to use

Use when a metabolite list (KEGG compound IDs converted via MetaboAnalyst ID Conversion) must be mapped to KEGG pathways to find where the selected metabolites occur and ranked by occurrence count.

## When NOT to use

- The input metabolites lack metabolite identifiers convertible to KEGG compound IDs, so MetaboAnalyst ID Conversion cannot produce the KEGG IDs KEGGREST requires
- The goal is pathway enrichment for gene/transcript identifiers (KEGGREST mapping here targets metabolite compound IDs, not gene IDs)
- A metabolite-level statistical test (e.g., Repeated Measures ANOVA, linear mixed models) is needed first — this skill operates on an already-selected metabolite list

## Inputs

- CSV/text list of metabolite identifiers converted via MetaboAnalyst ID Conversion (pasted into MeTEor's pathway enrichment step 2 field)
- Original metabolite identifiers (e.g., HMDB IDs or compound names) requiring KEGG compound-ID conversion

## Outputs

- enriched_pathways.csv with rows ordered by descending occurrence, each containing a pathway name, an occurrence count, and a KEGG pathway-map URL
- Interactive KEGG pathway-map URLs with the identified metabolites highlighted in red

## How to apply

First convert metabolite identifiers (e.g., HMDB or compound names) via the MetaboAnalyst ID Conversion service, since 'the IDs are needed to map the metabolites to known KEGG pathways'; note the MetaboAnalyst API workaround: paste the converted csv content into MeTEor's pathway enrichment step-2 field. Then load these converted metabolite identifiers in MeTEor's Pathway Enrichment module and trigger 'Get enriched pathways'; under the hood MeTEor uses KEGGREST (in R) to map each converted metabolite ID to the KEGG pathways in which it occurs. Rank the resulting pathways by the number (occurrence) of query metabolites present in each pathway. For each enriched pathway, construct a modified URL to the interactive KEGG pathway map such that the identified metabolites are highlighted in red. Write the ranked list — pathway names, occurrence counts, and interactive KEGG outlinks — to enriched_pathways.csv.

## Related tools

- **MeTEor** (R Shiny app whose Pathway Enrichment module orchestrates pathway mapping, ranking by occurrence, and export of enriched_pathways.csv) — https://github.com/scibiome/meteor
- **KEGGREST** (R client used to map converted metabolite IDs to the KEGG pathways in which they occur)
- **MetaboAnalyst** (External web service (ID Conversion at metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml) that converts metabolite identifiers into KEGG compound IDs)
- **KEGG** (Pathway database providing the pathway definitions and interactive pathway maps highlighted by modified URLs)

## Examples

```
library(MeTEor); meteor()
```

## Evaluation signals

- enriched_pathways.csv exists after the run
- Each output row contains a pathway entry with an occurrence count and a KEGG pathway-map URL
- Rows are ordered by descending occurrence of the selected metabolites
- Every queried metabolite appears in at least the pathway list entries whose occurrence count includes it (IDs were successfully mapped via KEGGREST)
- Opened KEGG pathway URLs show the identified metabolites highlighted in red

## Limitations

- MetaboAnalyst ID conversion relies on a web workaround: the MetaboAnalyst API is currently unavailable, so converted csv content must be manually pasted into MeTEor's pathway enrichment step 2 field
- Pathways are ranked purely by occurrence of query metabolites; no statistical over-representation p-value is described in the enrichment output
- Requires metabolite identifiers resolvable to KEGG compound IDs; IDs outside KEGG's compound space cannot be mapped

## Evidence

- [methods] Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence.: "Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence."
- [methods] IDs needed for KEGG mapping: "The IDs are needed to map the metabolites to known KEGG pathways."
- [methods] MetaboAnalyst API disclaimer: "**Disclaimer:** The API of MetaboAnalyst is currently not available, therefore we developed this workaround."
- [readme] README enrichment features: [evidence span withheld — anchor status: external_doc]
- [readme] Pathway Enrichment module: [evidence span withheld — anchor status: external_doc]
