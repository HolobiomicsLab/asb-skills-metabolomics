---
name: metabolite-gene-interaction-mapping
description: Use when you have a standalone list of disease-associated metabolites
  (e.g. 39 plasma lipids or 56 urine metabolites) and need to convert them into gene
  sets for over-representation analysis against GWAS subnetworks.
when_to_use_negative:
- When you have paired metabolomics-GWAS data with matched SNP-metabolite associations
  — this skill addresses the unpaired, standalone setting and is 'limited to genes
  with known metabolite interactions'
- When metabolites have no known interactions recorded in HMDB, since discovered associations
  are restricted to genes with known metabolite interactions
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: metGWAS 1.0
  role: R workflow (metGWAS 1.0_19 April.R) that performs the metabolite-to-gene mapping
    and downstream hypergeometric comparison
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Browser driver enabling the online HMDB database queries during metabolite-gene
    mapping
- name: qvalue
  role: R package for multiple-testing (FDR) correction of pathway enrichment P values
- name: R
  role: Environment in which the mapping workflow and supporting install scripts run
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:3f1ae7b666e181df2592f642de5ef7c3d9d3baafa9db8ee115267e68144cc7b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolite-gene-interaction-mapping

## Summary

Maps disease-associated metabolites to their interacting proteins/genes via online databases (HMDB), building the metabolic gene sets that metGWAS 1.0 compares against GWAS Catalog-derived genomic gene sets with a hypergeometric test.

## When to use

Use when you have a standalone list of disease-associated metabolites (e.g. 39 plasma lipids or 56 urine metabolites) and need to convert them into gene sets for over-representation analysis against GWAS subnetworks.

## When NOT to use

- When you have paired metabolomics-GWAS data with matched SNP-metabolite associations — this skill addresses the unpaired, standalone setting and is 'limited to genes with known metabolite interactions'
- When metabolites have no known interactions recorded in HMDB, since discovered associations are restricted to genes with known metabolite interactions

## Inputs

- List of disease-associated/standalone metabolites (e.g. 39 case-study-1 plasma lipidome metabolites or 56 case-study-2 urine metabolites)
- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (HMDB discoverable gene list, 64KB)
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (GWAS Catalog network gene sets, 64,516KB, from Harvard Dataverse doi:10.7910/DVN/B32FRP)
- RSelenium browser driver session for online HMDB queries

## Outputs

- Metabolic gene sets (metabolite-interacting genes, optionally grouped by KEGG pathway)
- Hypergeometric over-representation results with overlap size, gene identities, and P values (e.g. metGWAS_casestudy1_results.csv, metgwas_case2_enrichment_results.csv)
- qvalue-adjusted FDR values for pathway-level tests

## How to apply

Within metGWAS 1.0 (metGWAS 1.0_19 April.R), take the input metabolite list and query HMDB online — using the RSelenium browser driver set up by 'RSelenium Driver SetUp.R' — to retrieve the proteins/genes that interact with each metabolite. Restrict the resulting gene universe to the HMDB 'discoverable genes' background loaded from discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData so that downstream hypergeometric tests are properly normalized. For pathway-level analyses (e.g. case study 2's KEGG pathways), group metabolite-interacting genes by pathway before testing. The output metabolic gene sets are then compared against genomic gene sets extracted from the GWAS Catalog network representation (GWASnetwork_discoverableGenesOnly_2021_10_14.RData) via the hypergeometric over-representation test, with qvalue used for FDR correction (retain pathways at FDR ≤ 0.05).

## Related tools

- **metGWAS 1.0** (R workflow (metGWAS 1.0_19 April.R) that performs the metabolite-to-gene mapping and downstream hypergeometric comparison) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Browser driver enabling the online HMDB database queries during metabolite-gene mapping)
- **qvalue** (R package for multiple-testing (FDR) correction of pathway enrichment P values)
- **R** (Environment in which the mapping workflow and supporting install scripts run)

## Examples

```
Rscript "RSelenium Driver SetUp.R"; Rscript "metGWAS 1.0_19 April.R"  # maps metabolites to HMDB-interacting genes, then hypergeometric test vs GWAS Catalog network
```

## Evaluation signals

- Case study 1 validation: mapping the 39 plasma lipidome metabolites through the glycerophospholipid metabolism pathway must reproduce a 9-gene overlap (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) with the cardiovascular disease GWAS subnetwork at P = .0028 / P < 0.005 in metGWAS_casestudy1_results.csv
- Case study 2 validation: alanine, aspartate, and glutamate metabolism should yield 5 genes (AGXT2, CLYBL, EGLN3, PHF8, ASRGL1) overlapping the urine GWAS network at P = 0.13, and glycine, serine, and threonine metabolism 8 genes at P = 0.103 against the kidney network
- APOA5 must appear in the case study 1 overlap, matching the original metabolomics-GWAS study
- The gene universe must be restricted to the HMDB discoverable-gene list and the GWAS network to discoverable genes only (GWASnetwork_discoverableGenesOnly_2021_10_14.RData)
- Enriched KEGG pathways in case study 2 must survive qvalue FDR correction at FDR ≤ 0.05

## Limitations

- The GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516KB) is not included in the repository; it must be downloaded separately from Harvard Dataverse (doi:10.7910/DVN/B32FRP)
- Analysis is limited to genes with known metabolite interactions due to the unpaired nature of the datasets, so metabolites lacking HMDB interaction annotations contribute nothing
- The workflow depends on live online HMDB queries via RSelenium, requiring a working browser driver setup (RSelenium Driver SetUp.R)

## Evidence

- [methods] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [discussion] RSelenium Driver SetUp.R is required for online database queries: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] GWAS network RData file is too large for the repo and must be downloaded from Dataverse: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [intro] Analysis limited to genes with known metabolite interactions in the unpaired setting: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets, any discovered associations may represent biomarkers and druggable"
- [results] Case study 1 reproduced nine-gene glycerophospholipid metabolism overlap: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway and contain"
