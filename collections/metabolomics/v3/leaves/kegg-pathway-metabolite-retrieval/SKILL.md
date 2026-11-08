---
name: kegg-pathway-metabolite-retrieval
description: Use when MetaboAnalyst outputs ('name_map.csv' and 'pathway_results.csv')
  are available and KEGG pathways with FDR ≤ 0.05 must be populated with their member
  metabolites annotated as HMDB IDs.
when_to_use_negative:
- The input metabolite list already carries HMDB identifiers and its member metabolites
  per pathway, so no KEGG call-out is needed.
- No MetaboAnalyst 'pathway_results.csv' is available, or no KEGG pathway passes the
  FDR ≤ 0.05 threshold, leaving nothing to annotate.
- The analysis involves paired metabolomics-GWAS data rather than standalone datasets,
  which is outside the unpaired-data scope of metGWAS 1.0.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0154
tools:
- name: metGWAS 1.0
  role: Host R workflow whose Module 1 performs the pathway selection, KEGG retrieval,
    and HMDB annotation
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Drives the browser call-out to the KEGG database to fetch metabolite entries
    for each selected pathway; environment set up via RSelenium Driver SetUp.R
- name: qvalue
  role: Computes the FDR q-values used to filter pathway_results rows before metabolite
    retrieval; installed via qvalue and bioconductor install.R
- name: R
  role: Language in which the metGWAS 1.0 pipeline (metGWAS 1.0_19 April.R) is implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:dcfd86f6016d75988e7d4ecafb678b744bfbdd1151b0b2b2847f0bc72abad519
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kegg-pathway-metabolite-retrieval

## Summary

Module 1 of the metGWAS 1.0 R workflow: after MetaboAnalyst flags over-represented KEGG pathways by FDR, this skill retrieves the metabolite entries for each significant pathway via an RSelenium browser call-out to the KEGG database and maps them to HMDB IDs so downstream gene-set construction can proceed.

## When to use

Use when MetaboAnalyst outputs ('name_map.csv' and 'pathway_results.csv') are available and KEGG pathways with FDR ≤ 0.05 must be populated with their member metabolites annotated as HMDB IDs.

## When NOT to use

- The input metabolite list already carries HMDB identifiers and its member metabolites per pathway, so no KEGG call-out is needed.
- No MetaboAnalyst 'pathway_results.csv' is available, or no KEGG pathway passes the FDR ≤ 0.05 threshold, leaving nothing to annotate.
- The analysis involves paired metabolomics-GWAS data rather than standalone datasets, which is outside the unpaired-data scope of metGWAS 1.0.

## Inputs

- MetaboAnalyst name_map.csv (KEGG/HMDB metabolite identifier mapping)
- MetaboAnalyst pathway_results.csv (KEGg pathways with q-value/FDR from the qvalue package)
- FDR significance threshold (q-value ≤ 0.05)
- RSelenium-driven browser session to the KEGG database

## Outputs

- filtered_pathway_metabolites_hmdb.csv (HMDB metabolite identifiers annotated per significant KEGG pathway)

## How to apply

Load the MetaboAnalyst 'name_map.csv' and 'pathway_results.csv' tables into the R environment and filter pathway_results rows to those with FDR (q-value, computed via the qvalue package) ≤ 0.05. Set up the browser-automation environment with the provided 'RSelenium Driver SetUp.R' script, then for each significant pathway drive a call-out to the KEGG database to retrieve the metabolite entries belonging to that pathway. Map the retrieved KEGG metabolite entries to HMDB identifiers using the name_map table and write the annotated set of HMDB metabolites per significant pathway to filtered_pathway_metabolites_hmdb.csv. This annotation step is the basis for later mapping metabolites to interacting proteins/genes — e.g. the KEGG glycerophospholipid metabolism pathway metabolites from which the nine reported genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) are derived.

## Related tools

- **metGWAS 1.0** (Host R workflow whose Module 1 performs the pathway selection, KEGG retrieval, and HMDB annotation) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Drives the browser call-out to the KEGG database to fetch metabolite entries for each selected pathway; environment set up via RSelenium Driver SetUp.R)
- **qvalue** (Computes the FDR q-values used to filter pathway_results rows before metabolite retrieval; installed via qvalue and bioconductor install.R)
- **R** (Language in which the metGWAS 1.0 pipeline (metGWAS 1.0_19 April.R) is implemented)

## Examples

```
source("RSelenium Driver SetUp.R"); source("metGWAS 1.0_19 April.R")
```

## Evaluation signals

- filtered_pathway_metabolites_hmdb.csv contains only pathways whose pathway_results row has FDR ≤ 0.05.
- Every retained pathway row carries a non-empty annotation of HMDB metabolite identifiers (e.g. the KEGG glycerophospholipid metabolism pathway is annotated with its interacting metabolites).
- Each HMDB ID in the output is traceable back to a KEGG metabolite entry via the name_map.csv mapping.
- Downstream gene-set derivation succeeds: metabolites of the KEGG glycerophospholipid metabolism pathway map to the nine reported genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7).
- The RSelenium driver session initialized by RSelenium Driver SetUp.R completes each KEGG call-out without failed or empty retrievals.

## Limitations

- Annotation depends on a live RSelenium browser session and a call-out to the online KEGG database, so network/driver failures break the step; RSelenium Driver SetUp.R is provided specifically for environment setup.
- The large GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516KB) needed by the workflow is not included in the supplementary material and must be downloaded externally from the Harvard Dataverse link.
- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the datasets, so only metabolites with retrievable KEGG entries and HMDB mappings contribute.

## Evidence

- [other] Module 1 annotates the metabolites belonging to the chosen KEGG pathway through a call-out to the KEGG database, returning them as HMDB IDs.: [evidence span withheld — anchor status: not_found]
- [other] Filter the pathway results rows to those with FDR (q-value, computed via the qvalue package) ≤ 0.05.: [evidence span withheld — anchor status: not_found]
- [readme] The supplementary material-1 consists of several R-scripts (i.e., metGWAS 1.0_19 April, qvalue and bioconductor install, and RSelenium Driver SetUp): "The supplementary material-1 consists of several R-scripts (i.e., metGWAS 1.0_19 April, qvalue and bioconductor install, and RSelenium Driver SetUp)"
- [readme] The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [results] nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway: "nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway"
