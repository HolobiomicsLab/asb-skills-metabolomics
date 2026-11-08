---
name: kegg-pathway-id-resolution
description: Use when a metabolomics feature table contains non-standard identifiers
  (e.g., RI-coded names from GC retention time) that must be converted to standard
  compound IDs before KEGG pathway enrichment in MeTEor.
when_to_use_negative:
- Metabolites already carry standard database IDs (e.g., KEGG compound or HMDB identifiers)
  — conversion would be redundant
- The workflow requires fully automated, scripted identifier resolution — the MetaboAnalyst
  API is unavailable, so this workaround is manual and introduces a delay
- The identifiers are not metabolite/compound codes (e.g., gene or protein identifiers)
  — KEGGREST pathway mapping in MeTEor targets compounds
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalyst
  role: External ID conversion service used manually (web form) to convert RI-coded
    metabolite names to standard identifiers; API unavailable, so used as a workaround
- name: MeTEor
  role: R Shiny app whose Pathway Enrichment module consumes the converted IDs and
    maps compounds to KEGG pathways via KEGGREST
  repo: https://github.com/scibiome/meteor
- name: KEGGREST
  role: R package used inside MeTEor for the 'Map to KEGG Pathways' enrichment step
    after ID resolution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:382f7903e9dae26338285a3de6c5cb9160fc54960c20d9e1ecc58643c9944d12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - KEGGREST
  - MeTEor
  - MetaboAnalyst
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# kegg-pathway-id-resolution

## Summary

Resolves non-standard metabolite identifiers (e.g., retention-time-coded RI codes like RI1150.81, RI1532.53, RI1557.73) into standard database IDs that can be mapped to known KEGG pathways, using the MetaboAnalyst ID Conversion web service as a manual workaround because its API is unavailable.

## When to use

Use when a metabolomics feature table contains non-standard identifiers (e.g., RI-coded names from GC retention time) that must be converted to standard compound IDs before KEGG pathway enrichment in MeTEor.

## When NOT to use

- Metabolites already carry standard database IDs (e.g., KEGG compound or HMDB identifiers) — conversion would be redundant
- The workflow requires fully automated, scripted identifier resolution — the MetaboAnalyst API is unavailable, so this workaround is manual and introduces a delay
- The identifiers are not metabolite/compound codes (e.g., gene or protein identifiers) — KEGGREST pathway mapping in MeTEor targets compounds

## Inputs

- List of RI-coded metabolite identifiers (e.g., RI1150.81, RI1532.53, RI1557.73) from the metabolomics data set
- MetaboAnalyst ID Conversion submission form (ConvertView.xhtml) entry

## Outputs

- CSV table mapping each RI-coded metabolite to a standard database identifier
- Standardized compound IDs suitable for KEGG pathway mapping in MeTEor's Pathway Enrichment module

## How to apply

Compile the list of RI-coded metabolite names (e.g., RI1150.81, RI1532.53, RI1557.73) whose trajectories or enrichment you want to interpret. Because the MetaboAnalyst API is unavailable, open the MetaboAnalyst ID Conversion page (https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml), paste the metabolite names into the submission form, and click 'Submit' — this step is manual and introduces a slight delay in the workflow. On the result page, click the 'here' link at the bottom to export the mapped identifiers as a CSV. Inspect the mapping so each RI code is linked to a standard database ID, which MeTEor then uses to map metabolites to known KEGG pathways (via KEGGREST in its 'Map to KEGG Pathways' enrichment module). The rationale is that KEGG pathway mapping requires recognized compound IDs, so unresolved RI codes would silently drop features from the enrichment.

## Related tools

- **MetaboAnalyst** (External ID conversion service used manually (web form) to convert RI-coded metabolite names to standard identifiers; API unavailable, so used as a workaround)
- **MeTEor** (R Shiny app whose Pathway Enrichment module consumes the converted IDs and maps compounds to KEGG pathways via KEGGREST) — https://github.com/scibiome/meteor
- **KEGGREST** (R package used inside MeTEor for the 'Map to KEGG Pathways' enrichment step after ID resolution)

## Examples

```
Rscript -e "remotes::install_github('scibiome/meteor'); library(MeTEor); meteor()"  # then paste RI1150.81, RI1532.53, RI1557.73 into MetaboAnalyst ConvertView and export the mapped-IDs CSV
```

## Evaluation signals

- Every submitted RI-coded metabolite (e.g., RI1150.81, RI1532.53, RI1557.73) has a non-empty converted identifier in the exported output CSV
- The exported mapping table is in CSV format suitable for downstream KEGG pathway mapping
- Each RI code is linked to exactly one standard database ID usable for mapping to known KEGG pathways
- Converted IDs feed into MeTEor's 'Get enriched pathways' step without unmapped/failed features
- Pathway enrichment output lists pathways where the selected metabolites are present, ranked by occurrence

## Limitations

- The MetaboAnalyst API is currently unavailable, so ID conversion must be performed manually through the web form, introducing a slight delay in the analysis workflow
- Conversion depends on the retention-time-coded RI names being recognized by MetaboAnalyst; unrecognized codes would yield empty mappings
- MeTEor's changelog is not available ('No changelog found'), so service-side changes to the MetaboAnalyst conversion page may not be tracked

## Evidence

- [methods] MetaboAnalyst ID conversion requires manual web-form submission because its API is unavailable: "head to <a href='https://www.metaboanalyst.ca/MetaboAnalyst/upload/ConvertView.xhtml' target='_blank'>MetaboAnalyst</a> for the ID conversion"
- [methods] MetaboAnalyst API unavailability forces the manual workaround: "**Disclaimer:** The API of MetaboAnalyst is currently not available, therefore we developed this workaround."
- [methods] Converted IDs are needed for KEGG pathway mapping: "The IDs are needed to map the metabolites to known KEGG pathways."
- [readme] README confirms the enrichment chain: compound ID conversion via MetaboAnalyst, then KEGG pathway mapping via KEGGREST: [evidence span withheld — anchor status: external_doc]
- [methods] Pathway enrichment in MeTEor returns pathways ranked by occurrence: "Click 'Get enriched pathways' to receive a list of pathways where your selected metabolites are present, ranked by occurrence."
