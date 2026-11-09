---
name: gwas-catalog-network-gene-set-construction
description: 'Use when a metabolomics-only (standalone, unpaired) dataset needs to
  be compared against GWAS evidence: you must extract disease- or trait-anchored gene
  sets (e.g. cardiovascular disease, kidney/urine) from the GWAS Catalog network before
  running a hypergeometric enrichment test.'
when_to_use_negative:
- You have paired metabolomics-GWAS data (per-metabolite SNP associations); metGWAS
  is explicitly designed for unpaired, standalone studies and cannot leverage genotype-level
  pairing.
- Your genes of interest lack known metabolite interactions in HMDB — the pipeline
  restricts all sets to the discoverable-gene background, so such genes are excluded
  by design.
- You need GWAS summary statistics or SNP-level fine mapping rather than gene-set-level
  over-representation; this skill only yields gene sets from a network view of the
  GWAS Catalog.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: metGWAS 1.0
  role: R script (metGWAS 1.0_19 April.R) that loads the GWAS Catalog network RData
    artifact and extracts disease/phenotype-specific genomic gene sets for hypergeometric
    comparison against metabolic gene sets
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Runtime environment in which the GWAS Catalog network RData objects are loaded
    and the subnetwork gene sets are queried
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:8f834d17dfbae20c58e81555547c0c9d369518f1d02301167c8aaaca1895fc1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# gwas-catalog-network-gene-set-construction

## Summary

Construct disease/phenotype-specific genomic gene sets from a network representation of the GWAS Catalog (hundreds of studies), restricted to genes discoverable via known metabolite interactions in HMDB, for use as the genomic side of metGWAS 1.0's hypergeometric over-representation test.

## When to use

Use when a metabolomics-only (standalone, unpaired) dataset needs to be compared against GWAS evidence: you must extract disease- or trait-anchored gene sets (e.g. cardiovascular disease, kidney/urine) from the GWAS Catalog network before running a hypergeometric enrichment test.

## When NOT to use

- You have paired metabolomics-GWAS data (per-metabolite SNP associations); metGWAS is explicitly designed for unpaired, standalone studies and cannot leverage genotype-level pairing.
- Your genes of interest lack known metabolite interactions in HMDB — the pipeline restricts all sets to the discoverable-gene background, so such genes are excluded by design.
- You need GWAS summary statistics or SNP-level fine mapping rather than gene-set-level over-representation; this skill only yields gene sets from a network view of the GWAS Catalog.

## Inputs

- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516 KB), pre-built network representation of the GWAS Catalog restricted to discoverable genes (Harvard Dataverse doi:10.7910/DVN/B32FRP)
- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64 KB), HMDB discoverable-gene background universe
- A disease/phenotype anchor for subnetwork extraction (e.g. cardiovascular disease, kidney disease, urine-related traits)

## Outputs

- Disease/phenotype-specific genomic gene sets (gene lists) extracted from the GWAS Catalog network, e.g. the cardiovascular disease-related GWAS subnetwork or kidney-/urine-anchored subnetworks
- Gene overlap tables with hypergeometric P values, e.g. metGWAS_casestudy1_results.csv and metgwas_case2_enrichment_results.csv

## How to apply

Download the pre-built RData artifact GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516 KB) from the Harvard Dataverse deposit (doi:10.7910/DVN/B32FRP) — it is too large for the GitHub repo and only an external link is provided. Load it into R alongside discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData, which defines the background universe of discoverable genes (all genes in HMDB). Within metGWAS 1.0 (metGWAS 1.0_19 April.R), the network representation of the GWAS Catalog comprising 100s of studies is queried to extract genomic gene sets restricted to the disease or phenotype of interest (e.g. cardiovascular disease-associated studies, or kidney- and urine-anchored studies). Because the data are unpaired, associations are deliberately limited to genes with known metabolite interactions, so gene sets are intersected with the HMDB discoverable-gene background. These extracted genomic gene sets then serve as the comparison set for the hypergeometric over-representation test against metabolite-interacting metabolic gene sets.

## Related tools

- **metGWAS 1.0** (R script (metGWAS 1.0_19 April.R) that loads the GWAS Catalog network RData artifact and extracts disease/phenotype-specific genomic gene sets for hypergeometric comparison against metabolic gene sets) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Runtime environment in which the GWAS Catalog network RData objects are loaded and the subnetwork gene sets are queried) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
load('objects/GWASnetwork_discoverableGenesOnly_2021_10_14.RData'); load('objects/discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData'); source('metGWAS 1.0_19 April.R')
```

## Evaluation signals

- The downloaded GWASnetwork_discoverableGenesOnly_2021_10_14.RData loads without error and is 64,516 KB; the companion discoverableGenes RData is 64 KB.
- Case study 1 validation: the cardiovascular disease GWAS subnetwork overlaps the glycerophospholipid-metabolism gene set at P = 0.0028 with exactly nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7).
- Case study 2 validation: 8 genes at P = 0.103 for glycine, serine, and threonine metabolism vs the kidney network, and 5 genes (AGXT2, CLYBL, EGLN3, PHF8, ASRGL1) at P = 0.13 for alanine, aspartate, and glutamate metabolism vs the urine network.
- All extracted gene sets must be subsets of the HMDB discoverable-gene universe defined by discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData.
- APOA5 appears in the case study 1 overlap, matching the gene reported in the original metabolomics-GWAS study.

## Limitations

- The required RData file (GWASnetwork_discoverableGenesOnly_2021_10_14.RData, 64,516 KB) is not included in the repository or supplementary material; only an external Harvard Dataverse download link is provided.
- Because the metabolomics and GWAS data are unpaired, discovered associations are limited to genes with known metabolite interactions and are represented as gene-set-level enrichment, not SNP-level association.
- The GWAS Catalog network artifact is date-stamped (2021_10_14); results depend on that snapshot rather than a live GWAS Catalog query.

## Evidence

- [methods] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [discussion] RData artifact size and external hosting: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [intro] Discoverable-gene restriction rationale: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets, any discovered associations may represent biomarkers and druggable"
- [results] Case study 1 expected overlap: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway and contain"
- [readme] RData files and loading context: "1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)
2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
