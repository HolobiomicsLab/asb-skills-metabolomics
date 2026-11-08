---
name: hmdb-protein-interaction-retrieval
description: Use when a metabolomics result has been annotated with HMDB IDs and you
  must build the metabolite-interacting gene set (UniProt IDs → current human gene
  symbols) that metGWAS 1.0 needs for its hypergeometric over-representation test.
when_to_use_negative:
- Input metabolites lack HMDB IDs (e.g., only KEGG or InChIKey annotations), so their
  HMDB pages cannot be located by ID search.
- You already have paired metabolomics-GWAS data; this skill addresses the unpaired,
  standalone setting that is limited to genes with known metabolite interactions.
- The needed output is the genomic gene set from the GWAS Catalog network (GWASnetwork_discoverableGenesOnly_2021_10_14.RData),
  which this HMDB-retrieval module does not produce.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3489
- http://edamontology.org/topic_0121
tools:
- name: metGWAS 1.0
  role: R workflow whose Module 2 performs the HMDB lookup, UniProt-to-gene-symbol
    mapping, and writes metabolite_interacting_gene_set.csv
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Drives the automated browser session used to open each HMDB metabolite page
    and extract its interacting proteins (UniProt IDs)
- name: R
  role: Implementation language executing the scraping and UniProt-to-HGNC mapping
    steps in metGWAS 1.0_19 April.R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:ef2f0f7327d18acda41fd126c37af9e79ab52ac8651c8a24519e25ddede3c5f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3489
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hmdb-protein-interaction-retrieval

## Summary

Module 2 of the metGWAS 1.0 R workflow: it builds the metabolic gene set by searching each input metabolite's HMDB ID in the Human Metabolome Database (via RSelenium browser automation), harvesting the interacting proteins annotated as UniProt IDs, and mapping them to current human gene symbols for downstream hypergeometric over-representation analysis against GWAS Catalog-derived gene sets.

## When to use

Use when a metabolomics result has been annotated with HMDB IDs and you must build the metabolite-interacting gene set (UniProt IDs → current human gene symbols) that metGWAS 1.0 needs for its hypergeometric over-representation test.

## When NOT to use

- Input metabolites lack HMDB IDs (e.g., only KEGG or InChIKey annotations), so their HMDB pages cannot be located by ID search.
- You already have paired metabolomics-GWAS data; this skill addresses the unpaired, standalone setting that is limited to genes with known metabolite interactions.
- The needed output is the genomic gene set from the GWAS Catalog network (GWASnetwork_discoverableGenesOnly_2021_10_14.RData), which this HMDB-retrieval module does not produce.

## Inputs

- Annotated metabolite list with HMDB IDs (output of the upstream metabolite annotation step)
- Human Metabolome Database online metabolite pages listing interacting proteins as UniProt IDs
- RSelenium driver session set up by RSelenium Driver SetUp.R
- HMDB reference gene list discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (validation universe)

## Outputs

- metabolite_interacting_gene_set.csv — deduplicated table of unique current human gene symbols interacting with the input metabolites

## How to apply

Load the annotated metabolite list carrying HMDB IDs produced by the upstream annotation step. For each HMDB ID, use the RSelenium-driven browser session configured beforehand with RSelenium Driver SetUp.R to open the metabolite's page in the Human Metabolome Database and scrape the interacting proteins, which HMDB records as UniProt IDs. Within the metGWAS 1.0_19 April.R pipeline, map the harvested UniProt protein IDs to current human gene symbols using standard UniProt-to-HGNC identifier mapping and deduplicate the symbols. Write the metabolite-interacting gene set to metabolite_interacting_gene_set.csv; this metabolic gene set is what metGWAS 1.0 statistically compares to genomic gene sets from its GWAS Catalog network using a hypergeometric test. Validate the output against the HMDB reference universe discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData so only discoverable genes enter the over-representation analysis.

## Related tools

- **metGWAS 1.0** (R workflow whose Module 2 performs the HMDB lookup, UniProt-to-gene-symbol mapping, and writes metabolite_interacting_gene_set.csv) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Drives the automated browser session used to open each HMDB metabolite page and extract its interacting proteins (UniProt IDs))
- **R** (Implementation language executing the scraping and UniProt-to-HGNC mapping steps in metGWAS 1.0_19 April.R)

## Examples

```
source("RSelenium Driver SetUp.R"); source("metGWAS 1.0_19 April.R")
```

## Evaluation signals

- metabolite_interacting_gene_set.csv exists and parses as a table of unique current human gene symbols (no residual UniProt IDs, no duplicates).
- Every retrieved gene symbol falls within the HMDB discoverable-gene universe discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData used by metGWAS 1.0 for over-representation analysis.
- Each input HMDB ID yields at least one harvested UniProt ID; RSelenium page-load failures are surfaced rather than silently dropping metabolites.
- End-to-end sanity check (case study 1): intersecting the metabolic gene set with GWAS Catalog-derived gene sets recovers the nine glycerophospholipid-pathway genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) at P < 0.005.
- metGWAS 1.0's hypergeometric test runs without error using the retrieved set as the metabolic gene set against GWASnetwork_discoverableGenesOnly_2021_10_14.RData.

## Limitations

- Associations are limited to genes with known metabolite interactions in HMDB — the unpaired, standalone-data design cannot detect metabolite-gene links outside that discoverable universe.
- Depends on a live RSelenium browser session and the current HMDB page structure; driver setup failures or HMDB layout changes break the UniProt ID extraction.
- The validation universe is frozen to the 2021_10_14 HMDB snapshot (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData); newer metabolites or renamed gene symbols may fall outside the reference.

## Evidence

- [abstract] Metabolic gene sets are built by mapping metabolites to interacting proteins via online databases: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [discussion] RSelenium and qvalue/bioconductor scripts set up the R environment: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [intro] Analysis restricted to genes with known metabolite interactions (unpaired data): "limited to genes with known metabolite interactions due to the unpaired nature of the data sets"
- [readme] HMDB reference gene file used by the workflow: "1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)"
- [results] Case study 1 demonstrates metabolite-to-gene retrieval feeding pathway-level results: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway"
