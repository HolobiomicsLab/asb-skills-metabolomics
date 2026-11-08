---
name: metabolite-identifier-mapping
description: Use when metabolite features are named with retention-time codes (e.g.
  RI1150.81) or other non-standard identifiers and they must be converted to standard
  database IDs for KEGG pathway enrichment.
when_to_use_negative:
- Input metabolite names are already standard database IDs suitable for direct KEGGREST
  mapping (no conversion needed)
- A fully automated programmatic ID conversion is required — the MetaboAnalyst API
  is unavailable, so this skill relies on manual web-form submission and introduces
  a workflow delay
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalyst
  role: External ID Conversion service used manually (API unavailable) to map RI-coded
    metabolites to standard database identifiers, exported as CSV
- name: KEGGREST
  role: R package that maps the converted metabolite IDs to KEGG pathways and ranks
    pathways by occurrence of query metabolites
- name: MeTEor
  role: R Shiny app whose pathway enrichment module consumes the converted IDs and
    writes enriched_pathways.csv with KEGG pathway-map URLs highlighting metabolites
    in red
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:7d584aa6ec18716e76222c369cccb786c9804137f6dd27ba5cff029cd4354f64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - KEGGREST
  - MeTEor
  - MetaboAnalyst
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# metabolite-identifier-mapping

## Summary

Maps RI-coded metabolite identifiers (e.g., RI1150.81, RI1532.53, RI1557.73) to standard database IDs via MetaboAnalyst ID Conversion, so selected metabolites can be mapped to KEGG pathways in MeTEor's pathway enrichment step.

## When to use

Use when metabolite features are named with retention-time codes (e.g. RI1150.81) or other non-standard identifiers and they must be converted to standard database IDs for KEGG pathway enrichment.

## When NOT to use

- Input metabolite names are already standard database IDs suitable for direct KEGGREST mapping (no conversion needed)
- A fully automated programmatic ID conversion is required — the MetaboAnalyst API is unavailable, so this skill relies on manual web-form submission and introduces a workflow delay

## Inputs

- List of RI-coded metabolite identifiers (e.g. RI1150.81, RI1532.53, RI1557.73) selected in MeTEor
- MetaboAnalyst ID Conversion submission form (manual input)

## Outputs

- CSV file of mapped standard database identifiers exported from the MetaboAnalyst result page
- Converted metabolite IDs fed into MeTEor pathway enrichment (enriched_pathways.csv rows with pathway name, occurrence count, and interactive KEGG URL)

## How to apply

Compile the list of RI-coded metabolite names selected in MeTEor (e.g. RI1150.81, RI1532.53, RI1557.73). Because the MetaboAnalyst API is unavailable, open the MetaboAnalyst ID Conversion service at https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml as a manual workaround: paste the metabolite names into the submission form, click 'Submit', then click the 'here' link at the bottom of the result page to export the mapped identifiers as CSV. Paste the CSV content back into MeTEor's pathway enrichment step 2 field, where KEGGREST maps each converted ID to KEGG pathways and ranks them by occurrence of the query metabolites. Rationale: the standard IDs are required to map metabolites to known KEGG pathways and to construct interactive pathway-map URLs highlighting the metabolites in red.

## Related tools

- **MetaboAnalyst** (External ID Conversion service used manually (API unavailable) to map RI-coded metabolites to standard database identifiers, exported as CSV)
- **KEGGREST** (R package that maps the converted metabolite IDs to KEGG pathways and ranks pathways by occurrence of query metabolites)
- **MeTEor** (R Shiny app whose pathway enrichment module consumes the converted IDs and writes enriched_pathways.csv with KEGG pathway-map URLs highlighting metabolites in red) — https://github.com/scibiome/meteor

## Examples

```
R: remotes::install_github("scibiome/meteor"); library(MeTEor); meteor()  # then paste MetaboAnalyst-converted ID CSV into pathway enrichment step 2
```

## Evaluation signals

- Every submitted RI-coded metabolite has a non-empty converted identifier in the exported CSV
- The mapping output is in CSV format suitable for downstream KEGG pathway mapping
- enriched_pathways.csv rows each contain a pathway entry with an occurrence count and a KEGG pathway-map URL
- Rows in enriched_pathways.csv are ordered by descending occurrence of the selected metabolites
- Modified KEGG pathway URLs render the identified metabolites highlighted in red

## Limitations

- The MetaboAnalyst API is currently not available, so the ID conversion is a manual workaround that introduces a slight delay in the analysis workflow
- RI codes that lack a mapping in MetaboAnalyst will yield empty converted identifiers and cannot be used for KEGG pathway mapping

## Evidence

- [methods] Manual MetaboAnalyst workaround: "head to <a href='https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml' target='_blank'>MetaboAnalyst</a> for the ID conversion"
- [methods] API unavailable: "The API of MetaboAnalyst is currently not available, therefore we developed this workaround."
- [readme] Enrichment via KEGGREST: [evidence span withheld — anchor status: not_found]
- [methods] Purpose of converted IDs: "The IDs are needed to map the metabolites to known KEGG pathways."
