---
name: retention-index-metabolite-annotation
description: Use when metabolite features are named by retention-index codes (RI<value>
  format) from GC-based metabolomics and standard identifiers are required before
  KEGG pathway enrichment or compound mapping.
when_to_use_negative:
- Metabolite names are already standard database identifiers (e.g., KEGG compound
  IDs, HMDB) — conversion is redundant before KEGGREST pathway mapping.
- The analysis goal is pathway enrichment itself rather than identifier resolution
  — use MeTEor's KEGGREST-based 'Map to KEGG Pathways' step once valid IDs exist.
- The workflow requires fully automated, API-driven ID conversion — this skill is
  explicitly a manual workaround while the MetaboAnalyst API is unavailable.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalyst
  role: External ID Conversion service (ConvertView.xhtml) that maps RI-coded metabolite
    names to standard identifiers; used manually because its API is unavailable
- name: MeTEor
  role: R Shiny app whose workflow requires the converted IDs to map metabolites to
    known KEGG pathways for enrichment
  repo: https://github.com/scibiome/meteor
- name: KEGGREST
  role: Downstream R package used by MeTEor to map the converted compound IDs to KEGG
    pathways
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:0900d22670da6ed2e90fbc8fe0a3a5680e273132e7260ef007244d7da2c53778
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - KEGGREST
  - MeTEor
  - MetaboAnalyst
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# retention-index-metabolite-annotation

## Summary

Resolves retention-index-coded metabolite names (e.g., RI1150.81, RI1532.53, RI1557.73) into standard database identifiers via the MetaboAnalyst ID Conversion service so they can be mapped to KEGG pathways. Because the MetaboAnalyst API is unavailable, the resolution is a documented manual submission workaround.

## When to use

Use when metabolite features are named by retention-index codes (RI<value> format) from GC-based metabolomics and standard identifiers are required before KEGG pathway enrichment or compound mapping.

## When NOT to use

- Metabolite names are already standard database identifiers (e.g., KEGG compound IDs, HMDB) — conversion is redundant before KEGGREST pathway mapping.
- The analysis goal is pathway enrichment itself rather than identifier resolution — use MeTEor's KEGGREST-based 'Map to KEGG Pathways' step once valid IDs exist.
- The workflow requires fully automated, API-driven ID conversion — this skill is explicitly a manual workaround while the MetaboAnalyst API is unavailable.

## Inputs

- List of RI-coded metabolite identifiers (e.g., RI1150.81, RI1532.53, RI1557.73) from a GC metabolomics feature table
- MeTEor analysis session with metabolites of interest selected for pathway enrichment

## Outputs

- CSV file of mapped identifiers (exported via the 'here' link on the MetaboAnalyst result page)
- Standard database IDs per metabolite, suitable for KEGG pathway mapping

## How to apply

Compile the list of RI-coded metabolite names (e.g., RI1150.81, RI1532.53, RI1557.73) exported from the MeTEor analysis. Open the MetaboAnalyst ID conversion form (https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml) — MeTEor uses this as a manual workaround because the MetaboAnalyst API is currently unavailable, so this step introduces a delay and requires user input rather than an automated call. Paste the RI-coded names into the submission form and click 'Submit', then click the 'here' link at the bottom of the result page to export the mapping as CSV. Inspect the output so each RI code links to a standard database identifier that can feed KEGG compound mapping (via KEGGREST) and the MeTEor 'Get enriched pathways' step.

## Related tools

- **MetaboAnalyst** (External ID Conversion service (ConvertView.xhtml) that maps RI-coded metabolite names to standard identifiers; used manually because its API is unavailable)
- **MeTEor** (R Shiny app whose workflow requires the converted IDs to map metabolites to known KEGG pathways for enrichment) — https://github.com/scibiome/meteor
- **KEGGREST** (Downstream R package used by MeTEor to map the converted compound IDs to KEGG pathways)

## Evaluation signals

- Every submitted RI-coded metabolite has a non-empty converted identifier in the exported CSV.
- The mapping output is in CSV format suitable for downstream KEGG pathway mapping.
- Each RI code (e.g., RI1150.81, RI1532.53, RI1557.73) is linked to a standard database ID accepted by KEGGREST compound-to-pathway mapping.
- The 'Get enriched pathways' step in MeTEor returns a ranked pathway list rather than an empty result, indicating the converted IDs were valid.

## Limitations

- The MetaboAnalyst API is currently unavailable, so conversion requires manual pasting of metabolite names into the web form, introducing a delay in the analysis workflow.
- Converted identifiers must be inspected manually; ambiguous or unmapped RI codes will fail the non-empty-identifier validation before KEGG mapping.

## Evidence

- [methods] MetaboAnalyst API unavailable workaround: "**Disclaimer:** The API of MetaboAnalyst is currently not available, therefore we developed this workaround."
- [methods] KEGG ID requirement: "The IDs are needed to map the metabolites to known KEGG pathways."
- [readme] README enrichment features: [evidence span withheld — anchor status: external_doc]
