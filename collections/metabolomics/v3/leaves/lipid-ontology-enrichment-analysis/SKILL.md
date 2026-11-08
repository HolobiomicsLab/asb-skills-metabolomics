---
name: lipid-ontology-enrichment-analysis
description: Use when the query set is a list of lipid species (e.g., PA(18:0/16:0),
  TG(16:0/16:0/16:0), GlcCer(d18:0/24:0)) rather than KEGG metabolites, and enrichment
  should be tested against lipid ontology terms instead of KEGG pathways.
when_to_use_negative:
- The query set consists of KEGG metabolites and the analysis target is KEGG pathway
  enrichment — use EnrichMet's standard KEGG workflow instead of substituting LION
  annotations.
- The analysis requires ranked-list enrichment (MetSEA/fgsea) or network/topology-based
  impact analysis rather than categorical over-representation of lipids.
- No lipid-ontology annotations can be retrieved and no user-supplied lipid pathway-to-lipid
  table is available — the package does not redistribute LION data, so ORA cannot
  proceed.
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0089
tools:
- name: EnrichMet
  role: R package whose modular over-representation analysis (Fisher's Exact Test
    with Benjamini-Hochberg adjustment) is applied to the LION lipid-ontology mapping
    instead of KEGG pathways
  repo: https://github.com/biodatalab/enrichmet.git
- name: LION
  role: Source of lipid–ontology associations (pathway-to-lipid mappings) retrieved
    at runtime through the BioPortal/LION interface as the background annotation source
- name: R
  role: Runtime environment in which the EnrichMet ORA workflow is executed and dot
    plots are generated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:ff6287abbfbafada7b665ee7d3609f462356ef7f6b0eab438fdb57b73a17616e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - LION
  - R
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lipid-ontology-enrichment-analysis

## Summary

Extends EnrichMet's Fisher's Exact Test over-representation framework beyond KEGG-based metabolomics to lipidomics by using lipid–ontology associations from the LION database as the pathway-to-lipid annotation source, as demonstrated in the lung cancer lipidomics case study (Figure S2).

## When to use

Use when the query set is a list of lipid species (e.g., PA(18:0/16:0), TG(16:0/16:0/16:0), GlcCer(d18:0/24:0)) rather than KEGG metabolites, and enrichment should be tested against lipid ontology terms instead of KEGG pathways.

## When NOT to use

- The query set consists of KEGG metabolites and the analysis target is KEGG pathway enrichment — use EnrichMet's standard KEGG workflow instead of substituting LION annotations.
- The analysis requires ranked-list enrichment (MetSEA/fgsea) or network/topology-based impact analysis rather than categorical over-representation of lipids.
- No lipid-ontology annotations can be retrieved and no user-supplied lipid pathway-to-lipid table is available — the package does not redistribute LION data, so ORA cannot proceed.

## Inputs

- Character vector of lipid species names as query set (e.g., PA(18:0/16:0), TG(16:0/16:0/16:0), GlcCer(d18:0/24:0))
- LION lipid-ontology pathway-to-lipid mapping table obtained via BioPortal/LION interface
- User-supplied Pathway/Metabolites two-column annotation table (Pathway / Metabolites format)

## Outputs

- Over-representation results table with P_value, Adjusted_P_value, and Enrichment_Ratio per LION lipid-ontology term
- Dot plot of -log10(adjusted p-values) for enriched lipid-ontology pathways (Figure S2 analog)
- Set of enriched lipid-ontology terms at adjusted p-value < 0.05

## How to apply

Retrieve the LION lipid-ontology pathway-to-lipid mapping via the BioPortal/LION interface using the EnrichMet helper (e.g., fetch_lion_lipid_ontology()), which fetches content on demand and caches it locally (e.g., with BiocFileCache); note the package does not ship LION data. Reshape the mapping into the generic Pathway/Metabolites two-column table format that EnrichMet's over-representation interface accepts for any pathway–feature mapping. Supply the lipid species list as the query set and run over-representation analysis with Fisher's Exact Test on the 2x2 contingency table of lipids mapped to LION terms. Adjust p-values with Benjamini-Hochberg, treat adjusted p-value < 0.05 as significance, rank pathways by -log10(adjusted p-value), and visualize as dot plots corresponding to Figure S2. The rationale is that EnrichMet's modular ORA is agnostic to the annotation source, so swapping KEGG for LION demonstrates flexibility beyond metabolomics.

## Related tools

- **EnrichMet** (R package whose modular over-representation analysis (Fisher's Exact Test with Benjamini-Hochberg adjustment) is applied to the LION lipid-ontology mapping instead of KEGG pathways) — https://github.com/biodatalab/enrichmet.git
- **LION** (Source of lipid–ontology associations (pathway-to-lipid mappings) retrieved at runtime through the BioPortal/LION interface as the background annotation source)
- **R** (Runtime environment in which the EnrichMet ORA workflow is executed and dot plots are generated)

## Examples

```
PathwayVsLipids <- fetch_lion_lipid_ontology(); results <- enrichmet(inputMetabolites = lipid_list, PathwayVsMetabolites = PathwayVsLipids, input_type = "name")
```

## Evaluation signals

- Adjusted p-values are Benjamini-Hochberg corrected and enriched lipid-ontology terms pass adjusted p-value < 0.05, matching the case-study significance criterion
- Results and dot plot reproduce the lipidomics case-study output (Figure S2), demonstrating applicability beyond KEGG metabolomics
- The annotation table conforms to the Pathway/Metabolites two-column format accepted by the over-representation interface
- LION content is retrieved via the BioPortal/LION interface and cached locally (e.g., BiocFileCache) rather than shipped with the package
- All query lipid species in the input list are interpretable against the LION mapping (lipids unmapped to any ontology term should be accounted for in the contingency table)

## Limitations

- Lion lipid ontology content is not shipped with the package; it must be obtained at runtime via the BioPortal/LION programming interface, so the analysis depends on external API availability and local caching
- The framework extension was demonstrated only on a lipidomics dataset comparing control and mutant lung cancer samples; performance on other lipid lists or ontologies is not validated here
- Significant lipids for the case study were selected with thresholds (absolute fold change >= 1 and adjusted p-value < 0.05); different input thresholds will change the contingency table

## Evidence

- [intro] Lion lipid ontology mapping source: "Lipid–ontology associations of this analysis were sourced from the LION database [30]"
- [abstract] Flexible extension beyond KEGG: "its flexible framework supports extensions to other data types and knowledge bases beyond KEGG, as illustrated through a lipidomics case study"
- [readme] Lion runtime retrieval via BioPortal: "LION lipid ontology content is **not shipped with the package**. Instead, LION lipid ontology content is obtained at runtime through the BioPortal / LION programming interface"
- [readme] Generic pathway-feature table interface: "The same over-representation interface accepts any pathway–feature table in `Pathway` / `Metabolites` format."
- [intro] ORA method: "Fisher’s Exact Test [12] was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations"
