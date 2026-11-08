---
name: hmdb-annotation-of-pathway-metabolites
description: Use when MetaboAnalyst pathway_results.csv rows must be filtered to FDR
  ≤ 0.05 and each selected KEGG pathway's metabolites need resolving to HMDB IDs for
  downstream gene-set mapping in metGWAS 1.0.
when_to_use_negative:
- No pathway in pathway_results.csv passes the FDR ≤ 0.05 threshold — the selection
  step yields nothing and the KEGG call-out should not be run
- Input metabolomics data is not a MetaboAnalyst output set (no name_map.csv) — the
  KEGG-to-HMDB mapping central to this skill cannot be performed
- The goal is building genomic gene sets from the GWAS Catalog network rather than
  annotating the metabolites of selected KEGG pathways
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3407
tools:
- name: metGWAS 1.0
  role: 'R workflow hosting Module 1: loads the MetaboAnalyst tables, filters pathways
    at FDR ≤ 0.05, and writes filtered_pathway_metabolites_hmdb.csv (run via metGWAS
    1.0_19 April.R)'
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: drives the browser call-out to the KEGG database to retrieve the metabolite
    entries of each selected pathway; session set up via RSelenium Driver SetUp.R
- name: qvalue
  role: supplies the FDR q-values in pathway_results.csv used for the ≤ 0.05 pathway
    selection threshold (installed via the provided qvalue and bioconductor install.R)
- name: MetaboAnalyst
  role: upstream producer of the name_map.csv and pathway_results.csv inputs consumed
    by this annotation module
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:c814a464741e3a550eb6079c9275b4b5d4a1592fcaf173d6ed3352a241a2e062
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - MetaboAnalyst
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hmdb-annotation-of-pathway-metabolites

## Summary

Module 1 of the metGWAS 1.0 R workflow: after filtering MetaboAnalyst KEGG pathway over-representation results to FDR (q-value) ≤ 0.05, the metabolites of each significant pathway are retrieved from the KEGG database via an RSelenium browser call-out and mapped to HMDB IDs using the MetaboAnalyst name_map table. These per-pathway HMDB metabolite sets are the identifier space through which metGWAS 1.0 subsequently maps metabolites to interacting genes via the Human Metabolome Database.

## When to use

Use when MetaboAnalyst pathway_results.csv rows must be filtered to FDR ≤ 0.05 and each selected KEGG pathway's metabolites need resolving to HMDB IDs for downstream gene-set mapping in metGWAS 1.0.

## When NOT to use

- No pathway in pathway_results.csv passes the FDR ≤ 0.05 threshold — the selection step yields nothing and the KEGG call-out should not be run
- Input metabolomics data is not a MetaboAnalyst output set (no name_map.csv) — the KEGG-to-HMDB mapping central to this skill cannot be performed
- The goal is building genomic gene sets from the GWAS Catalog network rather than annotating the metabolites of selected KEGG pathways

## Inputs

- MetaboAnalyst 'name_map.csv' — metabolite name mapping table resolving KEGG metabolite entries to HMDB IDs
- MetaboAnalyst 'pathway_results.csv' — KEGG pathway over-representation results with FDR (q-value) per pathway

## Outputs

- filtered_pathway_metabolites_hmdb.csv — per-significant-pathway (FDR ≤ 0.05) lists of annotated HMDB metabolite identifiers
- HMDB ID sets per selected KEGG pathway (e.g. glycerophospholipid metabolism metabolites) feeding downstream HMDB-to-gene interaction mapping

## How to apply

Load the MetaboAnalyst outputs into the metGWAS 1.0 R environment: 'name_map.csv' (metabolite name mapping linking KEGG entries to HMDB IDs) and 'pathway_results.csv' (pathway over-representation results with FDR q-values computed via the qvalue package). Filter the pathway_results rows to FDR ≤ 0.05 — this q-value cutoff is the decision point determining which pathways get annotated. For each significant pathway, use the RSelenium Driver SetUp.R web-driver session to drive a browser call-out to the KEGG database and retrieve the metabolite entries belonging to that pathway. Map each retrieved KEGG metabolite entry to its HMDB ID through the name_map table and write the annotated HMDB metabolite set per significant pathway to filtered_pathway_metabolites_hmdb.csv. The rationale is that HMDB IDs are the currency of metGWAS 1.0's metabolic gene set generation, where disease-associated metabolites are mapped to interacting proteins (genes) via online databases — e.g. the KEGG glycerophospholipid metabolism pathway metabolites from which the nine cardiovascular-disease genes were derived.

## Related tools

- **metGWAS 1.0** (R workflow hosting Module 1: loads the MetaboAnalyst tables, filters pathways at FDR ≤ 0.05, and writes filtered_pathway_metabolites_hmdb.csv (run via metGWAS 1.0_19 April.R)) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (drives the browser call-out to the KEGG database to retrieve the metabolite entries of each selected pathway; session set up via RSelenium Driver SetUp.R)
- **qvalue** (supplies the FDR q-values in pathway_results.csv used for the ≤ 0.05 pathway selection threshold (installed via the provided qvalue and bioconductor install.R))
- **MetaboAnalyst** (upstream producer of the name_map.csv and pathway_results.csv inputs consumed by this annotation module)

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- filtered_pathway_metabolites_hmdb.csv contains only pathways with FDR (q-value) ≤ 0.05 — no sub-threshold pathway rows present
- Every selected pathway row carries a non-empty annotation of HMDB metabolite identifiers
- For the KEGG glycerophospholipid metabolism pathway, the annotated metabolites map to the nine reported genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7)
- Each KEGG metabolite entry retrieved via the RSelenium call-out resolves to an HMDB ID in the name_map table — no unmatched entries in the output
- Output schema is one annotated HMDB metabolite set per significant pathway row in filtered_pathway_metabolites_hmdb.csv

## Limitations

- The KEGG retrieval depends on a functioning RSelenium web-driver/browser session (RSelenium Driver SetUp.R) — an external dependency that can fail independently of the R pipeline
- Pathways with marginal significance but above the cutoff (e.g. case study 2's kidney-related glycine/serine/threonine and alanine/aspartate/glutamate metabolism pathways, P = 0.10 and P = 0.13) are not annotated by this module
- Downstream HMDB-to-gene mapping relies on prebuilt RData objects not fully in the repository: GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) is only available via an external Harvard Dataverse download link

## Evidence

- [abstract] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [discussion] The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway"
- [discussion] Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
