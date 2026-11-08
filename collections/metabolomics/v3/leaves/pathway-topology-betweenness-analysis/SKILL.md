---
name: pathway-topology-betweenness-analysis
description: Use when you have a standalone list of metabolite names (e.g. 56 urine
  metabolites from a metabolomics-GWAS case study) and need KEGG pathway over-representation
  P-values, FDR, and pathway-impact scores to connect them to disease GWAS genes.
when_to_use_negative:
- You have paired metabolomics-GWAS data with individual-level SNP–metabolite associations
  — metGWAS 1.0 is designed for standalone, unpaired datasets where associations are
  limited to genes with known metabolite interactions
- Your metabolite names cannot be mapped to HMDB identifiers, so the metabolite-to-interacting-protein
  (gene) crosswalk cannot be built
- Your question is purely about SNPs/genomic gene sets with no metabolite input —
  the GWAS-network step alone suffices and the MetaboAnalyst pathway-impact step is
  unnecessary
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: metGWAS 1.0
  role: R pipeline that wraps the MetaboAnalyst-style preprocessing, HMDB mapping,
    hypergeometric over-representation test and pathway-impact computation
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Execution environment running metGWAS 1.0_19 April.R and the environment-setup
    scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R)
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: Human Metabolome Database
  role: Reference used to map disease-associated metabolites to interacting proteins
    (genes) when generating metabolic gene sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:7fcf9667b2647ce479c64633b8eb8eb0aba421750bbc710c0c8e0cbe3e68554d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - Human Metabolome Database
  - R
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Pathway topology / betweenness-centrality over-representation analysis of metabolite sets

## Summary

Maps a standalone metabolite list to HMDB identifiers and runs MetaboAnalyst-style over-representation analysis (hypergeometric test) plus betweenness-centrality-based pathway impact against KEGG pathway gene sets, as implemented inside the metGWAS 1.0 R workflow.

## When to use

Use when you have a standalone list of metabolite names (e.g. 56 urine metabolites from a metabolomics-GWAS case study) and need KEGG pathway over-representation P-values, FDR, and pathway-impact scores to connect them to disease GWAS genes.

## When NOT to use

- You have paired metabolomics-GWAS data with individual-level SNP–metabolite associations — metGWAS 1.0 is designed for standalone, unpaired datasets where associations are limited to genes with known metabolite interactions
- Your metabolite names cannot be mapped to HMDB identifiers, so the metabolite-to-interacting-protein (gene) crosswalk cannot be built
- Your question is purely about SNPs/genomic gene sets with no metabolite input — the GWAS-network step alone suffices and the MetaboAnalyst pathway-impact step is unnecessary

## Inputs

- Standalone metabolomics dataset (metabolite name list with intensities), e.g. Supplementary Material S2 urine metabolite table (56 metabolites)
- HMDB reference / gene-set object discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)
- GWAS network gene-set object GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB, from Harvard Dataverse doi:10.7910/DVN/B32FRP)

## Outputs

- name_map.csv — metabolite-name-to-HMDB identifier crosswalk table
- pathway_results.csv — per-pathway over-representation statistics (hypergeometric P-value, FDR) and betweenness-centrality-based pathway impact
- Ranked list of over-represented KEGG pathways (e.g. Table 2: four pathways at FDR <= 0.05 for case study 2)

## How to apply

Load the metabolite table (e.g. the case study 2 urine metabolomics dataset from Supplementary Material S2) into an R session running metGWAS 1.0 and apply the platform's fixed MetaboAnalyst-based preprocessing (normalization/scaling hard-coded in the Section 2.1 script). Map processed metabolite names to Human Metabolome Database (HMDB) identifiers to build the metabolite-to-interacting-protein crosswalk, writing the crosswalk as name_map.csv. Run hypergeometric over-representation testing of the mapped metabolites against pathway gene sets and compute betweenness-centrality-based pathway impact, writing the statistics table as pathway_results.csv. Compare the resulting pathway P-values to the original metabolomics-GWAS study's reported associations (e.g. case study 2 expects marginal significance P = 0.10 and P = 0.13 for glycine/serine/threonine and alanine/aspartate/glutamate metabolism against kidney GWAS data) to judge concordance. The rationale is that hypergeometric testing over network-defined gene sets, combined with topology (betweenness centrality) impact, lets unpaired metabolomics and GWAS data be statistically compared without individual-level genotype-metabolite pairs.

## Related tools

- **metGWAS 1.0** (R pipeline that wraps the MetaboAnalyst-style preprocessing, HMDB mapping, hypergeometric over-representation test and pathway-impact computation) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Execution environment running metGWAS 1.0_19 April.R and the environment-setup scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R)) — https://github.com/saifurbd28/metGWAS-1.0
- **Human Metabolome Database** (Reference used to map disease-associated metabolites to interacting proteins (genes) when generating metabolic gene sets)

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- name_map.csv contains a complete crosswalk: every one of the 56 case study 2 urine metabolites is assigned an HMDB identifier
- pathway_results.csv reproduces marginal significance P = 0.10 for glycine, serine, and threonine metabolism and P = 0.13 for alanine, aspartate, and glutamate metabolism against kidney GWAS data
- The over-representation step returns four over-represented KEGG pathways at FDR <= 0.05 for the 56-metabolite urine set, matching Table 2
- The two RData objects (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData and GWASnetwork_discoverableGenesOnly_2021_10_14.RData, dated 2021_10_14) are present in the objects folder before running the workflow
- Case study 1 check: the analysis recovers the nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) interacting with metabolites in the KEGG glycerophospholipid metabolism pathway at P < 0.005

## Limitations

- Analysis is limited to genes with known metabolite interactions because the metabolomics and GWAS datasets are unpaired; no individual-level SNP–metabolite associations are inferred
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) is too large for the repository and must be downloaded separately from Harvard Dataverse (doi:10.7910/DVN/B32FRP)
- The workflow depends on external browser automation (RSelenium Driver SetUp.R) and Bioconductor/qvalue setup scripts, and no changelog is provided for the repository, so version drift may affect reproducibility

## Evidence

- [readme] Hypergeometric test and metabolic gene-set generation: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [methods] HMDB-based metabolite-to-gene mapping: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [readme] Case study 2 expected P-values: "we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways to GWAS data relating to kidney disease."
- [readme] Main workflow script: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [intro] Scope limitation to unpaired data: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets, any discovered associations may represent biomarkers and druggable"
- [supplementary] Required RData objects: "1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) 2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
