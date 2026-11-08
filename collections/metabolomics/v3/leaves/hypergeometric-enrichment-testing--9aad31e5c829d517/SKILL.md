---
name: hypergeometric-enrichment-testing
description: Use when a standalone (unpaired) metabolomics dataset of disease-associated
  metabolites must be statistically evaluated against GWAS Catalog-derived gene sets
  to find novel metabolite-gene associations via a hypergeometric test.
when_to_use_negative:
- The metabolomics and GWAS data are paired (from the same metabolomics-GWAS study)
  — metGWAS 1.0 is designed for evaluating independent, standalone studies
- The large GWAS network file GWASnetwork_discoverableGenesOnly_2021_10_14.RData has
  not been downloaded — it is not included in the supplementary material and the workflow
  cannot run without it
- The metabolite list has not been mapped to interacting proteins/genes — the hypergeometric
  comparison operates on metabolic gene sets derived via online databases, not raw
  metabolite names
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3388
- http://edamontology.org/topic_0622
tools:
- name: metGWAS 1.0
  role: Implements the hypergeometric comparison of metabolic and genomic gene sets;
    main workflow script metGWAS 1.0_19 April.R
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Language in which the metGWAS 1.0 pipeline is implemented and executed
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Driver set-up script used to run the metGWAS 1.0 workflow environment
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: Bioconductor/qvalue install script used to prepare the R environment before
    running the workflow
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:6029efc3c3d4feaa97c791b5bfa51b62f2661804c8a9bea2b1888e7ff4926b0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3388
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hypergeometric-enrichment-testing

## Summary

Run hypergeometric over-representation testing as implemented in metGWAS 1.0, which statistically compares metabolic gene sets (disease-associated metabolites mapped to interacting proteins) against genomic gene sets from a network representation of the GWAS Catalog.

## When to use

Use when a standalone (unpaired) metabolomics dataset of disease-associated metabolites must be statistically evaluated against GWAS Catalog-derived gene sets to find novel metabolite-gene associations via a hypergeometric test.

## When NOT to use

- The metabolomics and GWAS data are paired (from the same metabolomics-GWAS study) — metGWAS 1.0 is designed for evaluating independent, standalone studies
- The large GWAS network file GWASnetwork_discoverableGenesOnly_2021_10_14.RData has not been downloaded — it is not included in the supplementary material and the workflow cannot run without it
- The metabolite list has not been mapped to interacting proteins/genes — the hypergeometric comparison operates on metabolic gene sets derived via online databases, not raw metabolite names

## Inputs

- Standalone metabolomics dataset of disease-associated metabolites (e.g., case study 2 urine metabolite table with 56 metabolites from Supplementary Material S2)
- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) — HMDB discoverable-gene reference
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) — network representation of the GWAS Catalog (downloaded from Harvard Dataverse)
- GWAS disease/phenotype data from the GWAS Catalog for the comparison trait (e.g., kidney disease)

## Outputs

- Hypergeometric test statistics comparing metabolic and genomic gene sets
- Over-represented KEGG pathways at FDR <= 0.05 (e.g., glycine, serine, and threonine metabolism; alanine, aspartate, and glutamate metabolism; listed in Table 2)
- name_map.csv — crosswalk table of metabolite names mapped to HMDB identifiers
- pathway_results.csv — pathway statistics table including betweenness-centrality-based pathway impact

## How to apply

Set up the R environment with the two provided scripts (qvalue and bioconductor install.R, and RSelenium Driver SetUp.R), then place the required objects (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData, 64KB, and GWASnetwork_discoverableGenesOnly_2021_10_14.RData, 64,516KB, downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP) in the objects folder. Run metGWAS 1.0_19 April.R, which drives the workflow via RSelenium: the standalone metabolite list (e.g., the 56 urine metabolites of case study 2 from Supplementary Material S2) is mapped to HMDB identifiers, and a hypergeometric test compares the resulting metabolic gene sets against genomic gene sets from the network representation of the GWAS Catalog. For case study 2, expect marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways against kidney-related GWAS data, with four KEGG pathways over-represented at FDR <= 0.05. Associations are limited to genes with known metabolite interactions because the datasets are unpaired.

## Related tools

- **metGWAS 1.0** (Implements the hypergeometric comparison of metabolic and genomic gene sets; main workflow script metGWAS 1.0_19 April.R) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Language in which the metGWAS 1.0 pipeline is implemented and executed) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Driver set-up script used to run the metGWAS 1.0 workflow environment) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (Bioconductor/qvalue install script used to prepare the R environment before running the workflow) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript 'metGWAS 1.0_19 April.R'
```

## Evaluation signals

- Case study 2 sanity check: glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways show marginal significance (P = 0.10 and P = 0.13) against kidney-related GWAS data
- Over-representation output contains four KEGG pathways at FDR <= 0.05 for the 56-metabolite urine dataset, matching Table 2
- Case study 1 sanity check: nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) in the KEGG glycerophospholipid metabolism pathway associated with cardiovascular disease at P < 0.005, with APOA5 matched from the original study
- Environment invariant: both RData objects are present in the objects folder before running metGWAS 1.0_19 April.R
- name_map.csv successfully maps input metabolite names to HMDB identifiers before hypergeometric testing

## Limitations

- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the datasets
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData is too large to include in the supplementary material and must be downloaded from the external Harvard Dataverse link
- No changelog is available in the repository, so version drift of the workflow scripts cannot be tracked

## Evidence

- [abstract] We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test.: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [results] Marginal significance for kidney-related pathways in case study 2: "we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways"
- [discussion] Environment setup scripts required before running the workflow: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] Main workflow script identification: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
