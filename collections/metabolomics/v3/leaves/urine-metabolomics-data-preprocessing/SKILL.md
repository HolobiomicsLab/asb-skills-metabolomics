---
name: urine-metabolomics-data-preprocessing
description: Use when you have a standalone (unpaired) urine metabolomics dataset
  — e.g. a list of 56 named metabolites — that must be normalized and converted to
  HMDB identifiers before pathway over-representation against GWAS-derived gene sets.
when_to_use_negative:
- You have paired metabolomics-GWAS data with matched variant-metabolite statistics
  — metGWAS 1.0 is designed for standalone, unpaired datasets, and discovered associations
  are limited to genes with known metabolite interactions
- Your input is already a gene-level table or GWAS summary statistics rather than
  a metabolite list, since the preprocessing module expects named metabolites that
  can be mapped to HMDB identifiers
- Metabolite names cannot be matched to HMDB identifiers, so the crosswalk (name_map.csv)
  and downstream hypergeometric test against metabolic gene sets would be undefined
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metGWAS 1.0
  role: R pipeline containing the fixed MetaboAnalyst-based preprocessing module,
    HMDB name mapping, and hypergeometric over-representation analysis for standalone
    urine metabolomics datasets
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Implementation environment in which the metGWAS 1.0_19 April.R script is executed
    after the qvalue/bioconductor install and RSelenium Driver SetUp scripts prepare
    the environment
- name: RSelenium
  role: Driver setup script (RSelenium Driver SetUp.R) required to establish the web-automation
    environment the pipeline needs before preprocessing runs
- name: qvalue
  role: Bioconductor dependency installed via the provided 'qvalue and bioconductor
    install.R' script so FDR-adjusted statistics can be computed downstream
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:076f1155fc76e2edc194841fa286c603aa535a2661bb4efcaa6f63dad368ec25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# urine-metabolomics-data-preprocessing

## Summary

Fixed first step of the metGWAS 1.0 R pipeline that takes a standalone urine metabolomics list (56 metabolites from case study 2, Supplementary Material S2), applies the hard-coded MetaboAnalyst-based normalization/scaling, and maps metabolite names to HMDB identifiers to feed downstream hypergeometric pathway over-representation.

## When to use

Use when you have a standalone (unpaired) urine metabolomics dataset — e.g. a list of 56 named metabolites — that must be normalized and converted to HMDB identifiers before pathway over-representation against GWAS-derived gene sets.

## When NOT to use

- You have paired metabolomics-GWAS data with matched variant-metabolite statistics — metGWAS 1.0 is designed for standalone, unpaired datasets, and discovered associations are limited to genes with known metabolite interactions
- Your input is already a gene-level table or GWAS summary statistics rather than a metabolite list, since the preprocessing module expects named metabolites that can be mapped to HMDB identifiers
- Metabolite names cannot be matched to HMDB identifiers, so the crosswalk (name_map.csv) and downstream hypergeometric test against metabolic gene sets would be undefined

## Inputs

- Urine metabolomics dataset from Supplementary Material S2 (56 standalone metabolite names, case study 2)
- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB, HMDB gene list)
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB, GWAS network gene set, external Harvard Dataverse download)

## Outputs

- name_map.csv — metabolite-name-to-HMDB-identifier crosswalk table
- pathway_results.csv — hypergeometric over-representation statistics plus betweenness-centrality-based pathway impact per KEGG pathway

## How to apply

Load the case study 2 urine metabolomics dataset from Supplementary Material S2 into an R session running the metGWAS 1.0 pipeline (metGWAS 1.0_19 April.R with the objects folder containing discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData and the GWASnetwork RData file). Apply the tool's fixed MetaboAnalyst-based preprocessing module, whose normalization and scaling choices are hard-coded in the Section 2.1 script rather than user-tunable. Map the processed metabolite names to HMDB identifiers against the Human Metabolome Database reference and write the crosswalk table as name_map.csv, because the metabolic gene sets are built by mapping metabolites to interacting proteins via online databases. The downstream step then runs hypergeometric over-representation testing of the mapped metabolites against pathway gene sets with betweenness-centrality-based pathway impact, writing pathway_results.csv.

## Related tools

- **metGWAS 1.0** (R pipeline containing the fixed MetaboAnalyst-based preprocessing module, HMDB name mapping, and hypergeometric over-representation analysis for standalone urine metabolomics datasets) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Implementation environment in which the metGWAS 1.0_19 April.R script is executed after the qvalue/bioconductor install and RSelenium Driver SetUp scripts prepare the environment)
- **RSelenium** (Driver setup script (RSelenium Driver SetUp.R) required to establish the web-automation environment the pipeline needs before preprocessing runs)
- **qvalue** (Bioconductor dependency installed via the provided 'qvalue and bioconductor install.R' script so FDR-adjusted statistics can be computed downstream)

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- name_map.csv exists and contains a crosswalk from the 56 urine metabolite names to HMDB identifiers
- pathway_results.csv shows glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism with marginal significance (P = 0.10 and P = 0.13) against kidney-related GWAS data
- Four KEGG pathways are over-represented at FDR <= 0.05 for the 56 urine metabolites, matching Table 2 of the paper
- Environment preconditions met: qvalue installed and RSelenium driver running before metGWAS 1.0_19 April.R is executed
- The GWASnetwork_discoverableGenesOnly_2021_10_14.RData object (64,516KB) is loaded from the Harvard Dataverse link, not assumed present in the repository

## Limitations

- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the datasets
- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData file is too large for the repository and must be downloaded separately from Harvard Dataverse (doi:10.7910/DVN/B32FRP)
- Case study 2 pathway signals are only marginally significant (P = 0.10 and P = 0.13), not genome-wide or FDR-significant at the P-value level
- Normalization/scaling in the MetaboAnalyst-based module is hard-coded, so users cannot tune preprocessing parameters

## Evidence

- [results] we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways to GWAS data relating to kidney disease: "we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways to GWAS data relating to kidney disease"
- [methods] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [readme] Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [intro] Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets"
- [abstract] The metGWAS 1.0 tool was evaluated using standalone metabolomics datasets extracted from two metabolomics-GWAS case studies.: "The metGWAS 1.0 tool was evaluated using standalone metabolomics datasets extracted from two metabolomics-GWAS case studies."
