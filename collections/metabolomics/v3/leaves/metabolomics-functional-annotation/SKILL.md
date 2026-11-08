---
name: metabolomics-functional-annotation
description: Use when a metabolomics study has a list of identified metabolites (e.g.
  HMDB or KEGG compound IDs) and the goal is to map them to KEGG pathways, rank pathways
  by metabolite occurrence, and produce annotated pathway-map links.
when_to_use_negative:
- The identifiers are not metabolite compound IDs that MetaboAnalyst ID Conversion
  can translate to KEGG compound IDs (e.g. raw spectral features without annotation)
- The goal is gene or protein pathway enrichment rather than metabolite-to-KEGG-pathway
  mapping
- A statistical significance test per pathway is required — MeTEor ranks pathways
  by occurrence count of query metabolites, not by enrichment p-value
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MeTEor
  role: R Shiny app whose pathway enrichment module orchestrates metabolite ID mapping,
    KEGG pathway ranking, and interactive URL generation
  repo: https://github.com/scibiome/meteor
- name: KEGGREST
  role: R package used to map each converted metabolite ID to the KEGG pathways in
    which it occurs
- name: MetaboAnalyst
  role: External web service used for metabolite ID conversion to KEGG compound IDs,
    since its API is currently unavailable
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:298fac97fe034887a2249bd6ed6528a5b5f62e8a2f96483988fcb67cc2b07612
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - KEGGREST
  - MeTEor
  - MetaboAnalyst
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolomics-functional-annotation

## Summary

Annotate identified metabolites to KEGG pathways and generate interactive pathway maps highlighting the compounds of interest, as implemented in MeTEor's pathway enrichment module using MetaboAnalyst ID conversion and KEGGREST.

## When to use

Use when a metabolomics study has a list of identified metabolites (e.g. HMDB or KEGG compound IDs) and the goal is to map them to KEGG pathways, rank pathways by metabolite occurrence, and produce annotated pathway-map links.

## When NOT to use

- The identifiers are not metabolite compound IDs that MetaboAnalyst ID Conversion can translate to KEGG compound IDs (e.g. raw spectral features without annotation)
- The goal is gene or protein pathway enrichment rather than metabolite-to-KEGG-pathway mapping
- A statistical significance test per pathway is required — MeTEor ranks pathways by occurrence count of query metabolites, not by enrichment p-value

## Inputs

- List of metabolite identifiers (e.g. HMDB or compound names) from longitudinal metabolomics data
- MetaboAnalyst ID Conversion output (CSV content of converted KEGG compound IDs)
- Metabolomics feature table in MeTEor-compatible long format

## Outputs

- enriched_pathways.csv with pathway names, occurrence counts, and interactive KEGG pathway-map URLs
- Interactive KEGG pathway maps with identified metabolites highlighted in red

## How to apply

First convert the metabolite identifiers via MetaboAnalyst's ID Conversion service (used as a workaround because the MetaboAnalyst API is currently unavailable), pasting the converted CSV content back into MeTEor's pathway enrichment step 2 field. Use KEGGREST in R to map each converted KEGG compound ID to the KEGG pathways in which it occurs. Rank the resulting pathways by the number (occurrence) of query metabolites present in each pathway — MeTEor exposes this via the 'Get enriched pathways' button, which returns a list of pathways where the selected metabolites are present, ranked by occurrence. For each enriched pathway, construct a modified URL to the interactive KEGG pathway map so that the identified metabolites are highlighted in red. Write the ranked list, with pathway names, occurrence counts, and interactive KEGG outlinks, to enriched_pathways.csv.

## Related tools

- **MeTEor** (R Shiny app whose pathway enrichment module orchestrates metabolite ID mapping, KEGG pathway ranking, and interactive URL generation) — https://github.com/scibiome/meteor
- **KEGGREST** (R package used to map each converted metabolite ID to the KEGG pathways in which it occurs)
- **MetaboAnalyst** (External web service used for metabolite ID conversion to KEGG compound IDs, since its API is currently unavailable)

## Examples

```
library(MeTEor); meteor()  # then use pathway enrichment step 2 with MetaboAnalyst-converted IDs and 'Get enriched pathways'
```

## Evaluation signals

- enriched_pathways.csv exists and every row contains a pathway entry with an occurrence count and a KEGG pathway-map URL
- Rows are ordered by descending occurrence of the selected metabolites
- Each generated pathway-map URL highlights the identified metabolites in red on the interactive KEGG map
- Metabolite IDs successfully converted via MetaboAnalyst ID Conversion are accepted in pathway enrichment step 2

## Limitations

- The MetaboAnalyst API is currently not available, so ID conversion relies on a manual web-based workaround
- Pathway enrichment ranks pathways only by occurrence count of the query metabolites; no statistical enrichment test or p-value threshold is applied
- Mapping depends on KEGG coverage; metabolites without KEGG compound IDs cannot be annotated

## Evidence

- [methods] MeTEor queries metabolite IDs via MetaboAnalyst ID Conversion, then executes pathway analysis using KEGGREST and generates modified URLs to interactive KEGG pathways for the metabolites of interest.: [evidence span withheld — anchor status: not_found]
- [methods] Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence.: "Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence."
- [methods] The API of MetaboAnalyst is currently not available, therefore we developed this workaround.: "**Disclaimer:** The API of MetaboAnalyst is currently not available, therefore we developed this workaround."
- [readme] Enrichment: Map compounds (MetaboAnalyst ID Conversion), Map to KEGG Pathways (KEGGREST): [evidence span withheld — anchor status: external_doc]
