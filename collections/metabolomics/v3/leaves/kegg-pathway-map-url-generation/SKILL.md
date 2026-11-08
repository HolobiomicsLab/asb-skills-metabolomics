---
name: kegg-pathway-map-url-generation
description: Use when a metabolomics workflow has converted compound identifiers (e.g.
  HMDB IDs) to KEGG IDs via MetaboAnalyst ID Conversion and needs ranked pathway enrichment
  plus interactive pathway-map links highlighting the query metabolites.
when_to_use_negative:
- Metabolite identifiers are raw HMDB/KEGG-unmapped names — run MetaboAnalyst ID Conversion
  first, as the MetaboAnalyst API workaround is required before pathway mapping
- Goal is statistical pathway over-representation testing with p-values rather than
  MeTEor's occurrence-ranked pathway listing
- Data are not metabolomics feature trajectories processed in MeTEor (the enrichment
  module consumes MeTEor/MetaboAnalyst output)
edam_operation: http://edamontology.org/operation_4003
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: KEGGREST
  role: R package used to map each converted metabolite ID to the KEGG pathways in
    which it occurs
  repo: https://github.com/scibiome/meteor
- name: KEGG
  role: Source of pathway definitions and interactive pathway maps whose URLs are
    modified to highlight identified metabolites
- name: MetaboAnalyst
  role: ID conversion service producing the KEGG-compatible metabolite IDs that feed
    pathway enrichment
- name: MeTEor
  role: R Shiny app hosting the Pathway Enrichment module that clicks through to 'Get
    enriched pathways' ranked by occurrence
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:10e3782baf978997d5e510abd351b2d351a3961476efdbbdb829025785b621c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - KEGG
  - KEGGREST
  - MeTEor
  - MetaboAnalyst
  edam_operation: http://edamontology.org/operation_4003
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# kegg-pathway-map-url-generation

## Summary

Generates ranked KEGG pathway enrichment output with interactive KEGG map URLs in which the user's identified metabolites are highlighted, as implemented in MeTEor's Pathway Enrichment module via KEGGREST on MetaboAnalyst-converted metabolite IDs.

## When to use

Use when a metabolomics workflow has converted compound identifiers (e.g. HMDB IDs) to KEGG IDs via MetaboAnalyst ID Conversion and needs ranked pathway enrichment plus interactive pathway-map links highlighting the query metabolites.

## When NOT to use

- Metabolite identifiers are raw HMDB/KEGG-unmapped names — run MetaboAnalyst ID Conversion first, as the MetaboAnalyst API workaround is required before pathway mapping
- Goal is statistical pathway over-representation testing with p-values rather than MeTEor's occurrence-ranked pathway listing
- Data are not metabolomics feature trajectories processed in MeTEor (the enrichment module consumes MeTEor/MetaboAnalyst output)

## Inputs

- CSV of metabolite identifiers converted to KEGG compound IDs via MetaboAnalyst ID Conversion (pasted into pathway enrichment step 2 field)
- KEGGREST-mapped metabolite-to-pathway associations

## Outputs

- enriched_pathways.csv: ranked pathway list with pathway names, occurrence counts, and interactive KEGG pathway-map URLs with metabolites highlighted in red

## How to apply

Load the converted metabolite identifiers (the CSV content pasted back into MeTEor's pathway enrichment step 2 field) and use KEGGREST to map each converted ID to the KEGG pathways in which it occurs. Rank pathways by the number (occurrence) of query metabolites present — MeTEor returns the list ranked by occurrence. For each enriched pathway, construct a modified URL to the interactive KEGG pathway map such that the identified metabolites are highlighted in red. Write pathway names, occurrence counts, and the interactive KEGG outlinks to enriched_pathways.csv, ordered by descending occurrence. The rationale is that KEGG compound IDs are required, since 'The IDs are needed to map the metabolites to known KEGG pathways.'

## Related tools

- **KEGGREST** (R package used to map each converted metabolite ID to the KEGG pathways in which it occurs) — https://github.com/scibiome/meteor
- **KEGG** (Source of pathway definitions and interactive pathway maps whose URLs are modified to highlight identified metabolites)
- **MetaboAnalyst** (ID conversion service producing the KEGG-compatible metabolite IDs that feed pathway enrichment)
- **MeTEor** (R Shiny app hosting the Pathway Enrichment module that clicks through to 'Get enriched pathways' ranked by occurrence) — https://github.com/scibiome/meteor

## Examples

```
library(MeTEor); meteor()  # then load data, paste converted IDs into pathway enrichment step 2, and click 'Get enriched pathways'
```

## Evaluation signals

- enriched_pathways.csv exists and is non-empty
- Every row contains a pathway entry with an occurrence count and a KEGG pathway-map URL
- Rows are ordered by descending occurrence of the selected metabolites
- URLs point to interactive KEGG pathway maps and render the identified metabolites highlighted in red
- Each metabolite in the query set is traceable through MetaboAnalyst ID conversion to its KEGG pathway occurrences

## Limitations

- The MetaboAnalyst API is currently not available, so a manual web-based ID conversion workaround is used instead
- Ranking is purely by occurrence of query metabolites per pathway, not by statistical significance/p-values
- Interactive highlighting depends on KEGG pathway-map URL conventions and current KEGGREST availability

## Evidence

- [methods] Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence.: "Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence."
- [methods] IDs needed for mapping: "The IDs are needed to map the metabolites to known KEGG pathways."
- [methods] MetaboAnalyst workaround disclaimer: "**Disclaimer:** The API of MetaboAnalyst is currently not available, therefore we developed this workaround."
- [readme] README enrichment features: [evidence span withheld — anchor status: external_doc]
