---
name: manual-web-service-metabolite-query
description: Use when metabolite identifiers are retention-index-style codes (e.g.,
  RI1150.81) or other non-standard names that must be converted to standard database
  identifiers before KEGG pathway mapping, and no API is available.
when_to_use_negative:
- When the MetaboAnalyst API becomes available again — an automated call would avoid
  the manual delay this workaround introduces
- When metabolite identifiers are already standard database IDs (e.g., KEGG compound
  IDs) usable directly for KEGG pathway mapping
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalyst
  role: External ID Conversion web service used manually to map RI-coded metabolite
    names to standard identifiers
- name: MeTEor
  role: R Shiny app whose workflow requires the converted IDs before Pathway Enrichment
    (KEGG pathway mapping via KEGGREST)
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:7c545ec508e5070ac481fb9b46d26fa9218f52a510dc9659c605cfef4d38c9c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - MeTEor
  - MetaboAnalyst
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# manual-web-service-metabolite-query

## Summary

Manually converts RI-coded metabolite identifiers (e.g., RI1150.81, RI1532.53, RI1557.73) to standard database IDs via the MetaboAnalyst ID Conversion web form, as a workaround for the unavailable MetaboAnalyst API, so metabolites can be mapped to KEGG pathways.

## When to use

Use when metabolite identifiers are retention-index-style codes (e.g., RI1150.81) or other non-standard names that must be converted to standard database identifiers before KEGG pathway mapping, and no API is available.

## When NOT to use

- When the MetaboAnalyst API becomes available again — an automated call would avoid the manual delay this workaround introduces
- When metabolite identifiers are already standard database IDs (e.g., KEGG compound IDs) usable directly for KEGG pathway mapping

## Inputs

- List of RI-coded metabolite identifiers (e.g., RI1150.81, RI1532.53, RI1557.73) selected in MeTEor
- MetaboAnalyst ID Conversion web form (ConvertView.xhtml)

## Outputs

- CSV file of mapped identifiers downloaded via the 'here' link, linking each RI code to a standard database ID
- Standard metabolite IDs suitable for KEGG pathway mapping in MeTEor's Pathway Enrichment module

## How to apply

Compile the list of RI-coded metabolite names to be converted, then open the MetaboAnalyst conversion page (https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml). Because the MetaboAnalyst API is currently unavailable, MeTEor relies on this manual workaround: paste the metabolite names into the submission form and click 'Submit' to perform the ID conversion. Retrieve the mapping results by clicking the 'here' link at the bottom of the result page to export the mapped identifiers as CSV. Inspect the output so each RI code is linked to a standard database ID, which MeTEor then uses (together with KEGGREST) to map metabolites to known KEGG pathways during Pathway Enrichment.

## Related tools

- **MetaboAnalyst** (External ID Conversion web service used manually to map RI-coded metabolite names to standard identifiers)
- **MeTEor** (R Shiny app whose workflow requires the converted IDs before Pathway Enrichment (KEGG pathway mapping via KEGGREST)) — https://github.com/scibiome/meteor

## Examples

```
Open https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml, paste RI1150.81, RI1532.53, RI1557.73, click 'Submit', then export results via the 'here' link
```

## Evaluation signals

- Every submitted RI-coded metabolite has a non-empty converted identifier in the exported CSV
- The mapping results table is in CSV format suitable for downstream KEGG pathway mapping
- Each RI code (e.g., RI1150.81, RI1532.53, RI1557.73) is linked to exactly one standard database ID usable for KEGG mapping
- Converted IDs successfully feed into MeTEor's 'Get enriched pathways' step, returning a list of pathways ranked by occurrence

## Limitations

- The MetaboAnalyst API is currently unavailable, so the conversion must be performed manually in the web form, introducing a slight delay in the analysis workflow
- The conversion requires manual user input rather than an automated API call, making this step non-reproducible as scripted code

## Evidence

- [methods] API unavailable workaround: "**Disclaimer:** The API of MetaboAnalyst is currently not available, therefore we developed this workaround."
- [methods] Manual conversion step: "head to <a href='https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml' target='_blank'>MetaboAnalyst</a> for the ID conversion"
- [methods] IDs needed for KEGG mapping: "The IDs are needed to map the metabolites to known KEGG pathways."
- [readme] README enrichment features: [evidence span withheld — anchor status: external_doc]
