---
name: kegg-pathway-enrichment-analysis
description: Use when standalone metabolomics data must be linked to disease/phenotype
  GWAS results by testing whether metabolite-interacting genes from KEGG pathways
  are over-represented in GWAS gene networks.
when_to_use_negative:
- The metabolomics and GWAS datasets are paired from the same cohort — metGWAS 1.0
  targets standalone, independent studies; paired data support direct SNP–metabolite
  association testing instead.
- The metabolites cannot be mapped to HMDB interacting proteins, since the analysis
  is limited to genes with known metabolite interactions (HMDB 'discoverable genes'),
  leaving gene sets and the background universe too sparse to test.
- The trait of interest has no disease/phenotype gene set in the GWAS Catalog network
  representation to compare against.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0622
tools:
- name: metGWAS 1.0
  role: Runs the hypergeometric over-representation test comparing KEGG-pathway metabolic
    gene sets against GWAS-Catalog genomic gene sets via the script 'metGWAS 1.0_19
    April.R'
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: Bioconductor package used for multiple-testing correction; enriched KEGG pathways
    are retained at FDR ≤ 0.05
- name: RSelenium
  role: Browser driver used to access HMDB online so disease-associated metabolites
    can be mapped to interacting proteins (genes)
- name: R
  role: Execution environment for the environment-setup scripts and the metGWAS workflow
    script
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:92f0ebcbd1e5d8ab4a1dd63963c4f1569ba40467f8c99a4ac21a8072c41a0884
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kegg-pathway-enrichment-analysis

## Summary

Network-driven over-representation analysis of KEGG-pathway metabolic gene sets (metabolite-interacting genes mapped via HMDB) against GWAS-Catalog-derived genomic gene sets using a hypergeometric test, as implemented in the metGWAS 1.0 R workflow.

## When to use

Use when standalone metabolomics data must be linked to disease/phenotype GWAS results by testing whether metabolite-interacting genes from KEGG pathways are over-represented in GWAS gene networks.

## When NOT to use

- The metabolomics and GWAS datasets are paired from the same cohort — metGWAS 1.0 targets standalone, independent studies; paired data support direct SNP–metabolite association testing instead.
- The metabolites cannot be mapped to HMDB interacting proteins, since the analysis is limited to genes with known metabolite interactions (HMDB 'discoverable genes'), leaving gene sets and the background universe too sparse to test.
- The trait of interest has no disease/phenotype gene set in the GWAS Catalog network representation to compare against.

## Inputs

- Disease-associated metabolite list (e.g., 56 urine metabolites from the healthy-subject kidney-metabolism study)
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData — GWAS Catalog network representation (64,516 KB, Harvard Dataverse doi:10.7910/DVN/B32FRP)
- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData — HMDB discoverable-gene background universe (64 KB)
- KEGG pathway definitions (e.g., KEGG glycerophospholipid metabolism pathway; alanine, aspartate, and glutamate metabolism)
- Disease/phenotype genomic gene sets extracted from the GWAS Catalog network (e.g., kidney- or urine-anchored subnetworks)

## Outputs

- Per-pathway hypergeometric P-values for the overlap between metabolite-interacting genes and GWAS subnetwork genes
- Overlapping gene symbols per KEGG pathway (e.g., AGXT2, CLYBL, EGLN3, PHF8, ASRGL1 for alanine, aspartate, and glutamate metabolism; APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7 for glycerophospholipid metabolism)
- qvalue/FDR-corrected enrichment results table (e.g., metgwas_case2_enrichment_results.csv)

## How to apply

Set up the R environment with the supplied scripts 'qvalue and bioconductor install.R' and 'RSelenium Driver SetUp.R', then load GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516 KB, downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP) together with discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64 KB) to establish the GWAS Catalog network and the HMDB discoverable-gene background universe. In 'metGWAS 1.0_19 April.R', map the disease-associated metabolites (e.g., the 56 urine metabolites of the kidney-metabolism case study 2) to interacting proteins (genes) via HMDB using the RSelenium driver, and assemble metabolic gene sets for the KEGG pathways of interest. Extract the disease/phenotype genomic gene set from the network representation of the GWAS Catalog (e.g., kidney- or urine-anchored subnetworks). Run the hypergeometric over-representation test comparing each KEGG-pathway metabolic gene set against the genomic gene set, with all counts restricted to the discoverable-gene background. Apply multiple-testing correction with the qvalue package and retain enriched pathways at FDR ≤ 0.05 (four KEGG pathways in case study 2). Validate against published expectations: P < 0.005 for nine glycerophospholipid-metabolism genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) in case study 1, and marginal P = 0.10 (glycine, serine, threonine metabolism) and P = 0.13 (alanine, aspartate, glutamate metabolism; 5 genes including AGXT2) in case study 2.

## Related tools

- **metGWAS 1.0** (Runs the hypergeometric over-representation test comparing KEGG-pathway metabolic gene sets against GWAS-Catalog genomic gene sets via the script 'metGWAS 1.0_19 April.R') — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (Bioconductor package used for multiple-testing correction; enriched KEGG pathways are retained at FDR ≤ 0.05)
- **RSelenium** (Browser driver used to access HMDB online so disease-associated metabolites can be mapped to interacting proteins (genes))
- **R** (Execution environment for the environment-setup scripts and the metGWAS workflow script)

## Examples

```
Rscript "qvalue and bioconductor install.R"; Rscript "RSelenium Driver SetUp.R"; Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- Case study 1 reproduction: nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) recovered for the KEGG glycerophospholipid metabolism pathway at P < 0.005, with APOA5 matching the original metabolomics-GWAS study hit.
- Case study 2 reproduction: marginal P = 0.10 for glycine, serine, and threonine metabolism (8 genes vs kidney network) and P = 0.13 for alanine, aspartate, and glutamate metabolism (5 genes including AGXT2 vs urine network).
- Background invariant: overlap counts and P-values must be computed against the HMDB discoverable-gene universe (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData), not all protein-coding genes.
- Multiple-testing gate: only pathways passing qvalue correction at FDR ≤ 0.05 are reported as enriched (four KEGG pathways in case study 2); marginal hits (P = 0.10–0.13) are flagged as marginal, not significant.
- Output schema check: the enrichment table (e.g., metgwas_case2_enrichment_results.csv) lists pathway names, overlapping gene symbols, overlap counts, raw hypergeometric P-values, and FDR.

## Limitations

- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the data sets; genes without HMDB interaction annotations cannot be discovered.
- The design bridges standalone/independent metabolomics and GWAS studies, so it cannot confirm SNP–metabolite associations within a single cohort.
- The 64,516 KB GWASnetwork_discoverableGenesOnly_2021_10_14.RData is too large to be bundled in the repository and must be downloaded from the Harvard Dataverse deposit, so the workflow fails without network access or if the deposit link changes.
- Results can be marginal rather than significant: case study 2 reached only P = 0.10 and P = 0.13, which do not pass FDR ≤ 0.05.

## Evidence

- [abstract] We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test.: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [abstract] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [intro] Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets"
- [results] we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways: "we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways"
- [readme] The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
