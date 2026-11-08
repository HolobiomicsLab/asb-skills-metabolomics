---
name: metabolite-identifier-mapping
description: Use when a metabolomics-to-genomics pipeline needs to convert KEGG pathway
  metabolites into HMDB IDs and then into UniProt proteins/HGNC gene symbols, e.g.
  to build a metabolite-interacting gene set for over-representation analysis.
when_to_use_negative:
- Input metabolites lack HMDB identifiers or a name_map table, so KEGG-to-HMDB translation
  cannot be performed via the name_map lookup
- You have paired metabolomics-GWAS data (direct genotype-metabolite associations)
  rather than standalone datasets needing identifier-level bridging
- The target gene identifiers are already in hand and you only need the statistical
  hypergeometric comparison, not the mapping step
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metGWAS 1.0
  role: R workflow that orchestrates the KEGG->HMDB->UniProt->gene symbol mapping
    chain and writes the annotated output CSVs
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Drives browser call-outs to KEGG and HMDB pages to retrieve metabolite entries
    and interacting proteins
- name: qvalue
  role: Provides FDR q-values used to filter significant KEGG pathways before annotation
- name: R
  role: Implementation environment for the mapping pipeline, including UniProt-to-HGNC
    gene symbol mapping
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:1e6bba99c59504c94d39fded5bcc81ccca16fbc8a046086e1447af887f791b69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metabolite-identifier-mapping

## Summary

Chain cross-database identifier translation used by metGWAS 1.0: KEGG metabolite entries are annotated to HMDB IDs via a MetaboAnalyst name_map table, then HMDB IDs are resolved to interacting UniProt proteins and finally to current human gene symbols.

## When to use

Use when a metabolomics-to-genomics pipeline needs to convert KEGG pathway metabolites into HMDB IDs and then into UniProt proteins/HGNC gene symbols, e.g. to build a metabolite-interacting gene set for over-representation analysis.

## When NOT to use

- Input metabolites lack HMDB identifiers or a name_map table, so KEGG-to-HMDB translation cannot be performed via the name_map lookup
- You have paired metabolomics-GWAS data (direct genotype-metabolite associations) rather than standalone datasets needing identifier-level bridging
- The target gene identifiers are already in hand and you only need the statistical hypergeometric comparison, not the mapping step

## Inputs

- MetaboAnalyst 'name_map.csv' (metabolite name to HMDB ID mapping table)
- MetaboAnalyst 'pathway_results.csv' (KEGG pathway results with FDR/q-values)
- List of HMDB metabolite identifiers from annotated significant KEGG pathways
- RSelenium-driven browser session (per RSelenium Driver SetUp.R)

## Outputs

- filtered_pathway_metabolites_hmdb.csv (HMDB-annotated metabolites per significant KEGG pathway)
- metabolite_interacting_gene_set.csv (unique current human gene symbols mapped from metabolite-interacting UniProt proteins)

## How to apply

Load the MetaboAnalyst outputs 'name_map.csv' and 'pathway_results.csv' into the metGWAS 1.0 R environment and filter pathway rows to FDR (q-value via the qvalue package) <= 0.05. For each significant pathway (e.g. KEGG glycerophospholipid metabolism), use RSelenium-driven browser automation to call out to the KEGG database, retrieve that pathway's metabolite entries, and map them to HMDB IDs through the name_map table, writing filtered_pathway_metabolites_hmdb.csv. Next, for each HMDB ID, scrape the HMDB metabolite page (again via RSelenium) to extract interacting proteins annotated as UniProt IDs, then convert UniProt accessions to current human gene symbols by standard UniProt-to-HGNC identifier mapping inside the R pipeline. Deduplicate the gene symbols and write metabolite_interacting_gene_set.csv; the rationale is that metabolic gene sets are built by mapping metabolites to interacting proteins via online databases (KEGG, HMDB) so they can be compared to GWAS-derived genomic gene sets with a hypergeometric test.

## Related tools

- **metGWAS 1.0** (R workflow that orchestrates the KEGG->HMDB->UniProt->gene symbol mapping chain and writes the annotated output CSVs) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Drives browser call-outs to KEGG and HMDB pages to retrieve metabolite entries and interacting proteins)
- **qvalue** (Provides FDR q-values used to filter significant KEGG pathways before annotation)
- **R** (Implementation environment for the mapping pipeline, including UniProt-to-HGNC gene symbol mapping) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- filtered_pathway_metabolites_hmdb.csv contains only pathways with FDR <= 0.05 (q-value from the qvalue package)
- Each selected pathway row carries a non-empty annotation of HMDB metabolite identifiers (e.g. KEGG glycerophospholipid metabolism)
- metabolite_interacting_gene_set.csv parses as a table of unique current human gene symbols
- The gene set is consistent with the HMDB reference gene list (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData)
- Case study 1 sanity check: the nine genes APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7 should be recoverable from glycerophospholipid metabolism metabolite interactions

## Limitations

- Mapping depends on web-scraping KEGG and HMDB via RSelenium browser automation, so it breaks if the online database pages or driver setup change
- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the datasets
- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData object (64,516KB) is not in the repo and must be downloaded from the Harvard Dataverse link
- No changelog is present in the repository, so version drift of mapping behavior is hard to track

## Evidence

- [abstract] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [discussion] RSelenium/qvalue setup scripts: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] Main workflow script: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
