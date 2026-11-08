---
name: browser-automation-kegg-query
description: Use when a metabolomics over-representation workflow has identified KEGG
  pathways of interest (e.g. FDR ≤ 0.05) and you need the metabolite members of those
  pathways fetched from KEGG and cross-referenced to HMDB identifiers.
when_to_use_negative:
- The pathway already has complete member metabolite IDs locally or via a bulk KEGG
  REST API download, so no browser automation is needed.
- The pathway results have not been FDR-filtered (q-value > 0.05), so Module 1's significant-pathway
  selection criterion is not met.
- Input identifiers are HMDB-only with no KEGG counterpart in the name_map table,
  making the KEGG query and reverse mapping moot.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2640
tools:
- name: metGWAS 1.0
  role: R workflow whose Module 1 hosts the KEGG browser call-out and HMDB annotation
    step
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Drives the browser automation used to query the KEGG database for pathway
    metabolite entries
- name: R
  role: Implementation environment for the metGWAS 1.0 pipeline and Module 1 scripting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:1357afd77dd69708f881fe8f2cc04c05665ffe35a2bc5ce0cb3a2f0329fe4900
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# browser-automation-kegg-query

## Summary

An RSelenium-driven browser call-out to the KEGG database used inside metGWAS 1.0's Module 1 to retrieve the metabolite entries belonging to each significant KEGG pathway and return them as HMDB IDs.

## When to use

Use when a metabolomics over-representation workflow has identified KEGG pathways of interest (e.g. FDR ≤ 0.05) and you need the metabolite members of those pathways fetched from KEGG and cross-referenced to HMDB identifiers.

## When NOT to use

- The pathway already has complete member metabolite IDs locally or via a bulk KEGG REST API download, so no browser automation is needed.
- The pathway results have not been FDR-filtered (q-value > 0.05), so Module 1's significant-pathway selection criterion is not met.
- Input identifiers are HMDB-only with no KEGG counterpart in the name_map table, making the KEGG query and reverse mapping moot.

## Inputs

- MetaboAnalyst 'name_map.csv' table (KEGG-to-HMDB metabolite identifier mapping)
- MetaboAnalyst 'pathway_results.csv' table with FDR/q-value columns
- List of significant KEGG pathways selected at FDR ≤ 0.05

## Outputs

- filtered_pathway_metabolites_hmdb.csv — per significant pathway, an annotation of HMDB metabolite identifiers
- HMDB metabolite sets per KEGG pathway, ready for gene-interaction mapping

## How to apply

Load MetaboAnalyst's 'name_map.csv' and 'pathway_results.csv' tables into R and filter the pathway results rows to those with FDR (q-value, computed via the qvalue package) ≤ 0.05. For each retained significant pathway, use RSelenium to drive a browser call-out to the KEGG database and retrieve the metabolite entries belonging to that pathway. Map the retrieved KEGG metabolite entries to HMDB IDs via the name_map table and write the per-pathway annotated HMDB metabolite sets to filtered_pathway_metabolites_hmdb.csv. The RSelenium browser driver must be set up beforehand using the provided RSelenium Driver SetUp.R script, and the resulting annotated metabolite sets feed downstream mapping to interacting genes (e.g. the nine genes derived from the KEGG glycerophospholipid metabolism pathway metabolites in case study 1).

## Related tools

- **metGWAS 1.0** (R workflow whose Module 1 hosts the KEGG browser call-out and HMDB annotation step) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Drives the browser automation used to query the KEGG database for pathway metabolite entries)
- **R** (Implementation environment for the metGWAS 1.0 pipeline and Module 1 scripting)

## Examples

```
Rscript metGWAS_1.0_19_April.R  # Module 1: after filtering pathway_results.csv to FDR <= 0.05, RSelenium queries KEGG per pathway and writes filtered_pathway_metabolites_hmdb.csv
```

## Evaluation signals

- Output contains only pathways with FDR ≤ 0.05 (q-value filter enforced before annotation)
- Each selected pathway row carries a non-empty annotation of HMDB metabolite identifiers
- Retrieved KEGG metabolite entries map one-to-one onto HMDB IDs via the name_map table with no unmatched rows for annotated metabolites
- For the KEGG glycerophospholipid metabolism pathway, the annotated metabolite set supports derivation of the reported nine interacting genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7)
- RSelenium driver session was successfully initialized (RSelenium Driver SetUp.R completed) before the call-out

## Limitations

- The KEGG call-out depends on RSelenium browser automation and a live browser driver, which can break if the driver setup script or KEGG web interface changes.
- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the data sets, so annotated metabolite sets only yield discoverable genes.
- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData object (64,516KB) is not included in the repository; it must be downloaded externally from the Harvard Dataverse link.

## Evidence

- [other] After the KEGG pathway(s) of interest are selected by FDR threshold, Module 1 annotates the metabolites belonging to the chosen KEGG pathway through a call-out to the KEGG database, returning them as HMDB IDs.: [evidence span withheld — anchor status: not_found]
- [discussion] RSelenium Driver SetUp: "The supplementary material-1 consists of several R-scripts (i.e., metGWAS 1.0_19 April, qvalue and bioconductor install, and RSelenium Driver SetUp)"
- [results] KEGG glycerophospholipid metabolism pathway genes: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway"
- [readme] RSelenium driver setup script in repository: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
