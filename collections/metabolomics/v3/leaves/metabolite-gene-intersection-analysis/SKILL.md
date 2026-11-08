---
name: metabolite-gene-intersection-analysis
description: Use when you have an intersection gene set from a metabolic-vs-genomic
  hypergeometric comparison and need to determine, gene by gene, whether each gene
  sits at a primary keyword node or a first-neighbour node in the GWAS subnetwork.
when_to_use_negative:
- When the goal is to generate or statistically test the metabolic vs genomic gene
  sets themselves — that upstream step is metGWAS 1.0's hypergeometric test, not this
  provenance classification.
- When the metabolite-gene associations are paired (matched from the same metabolomics-GWAS
  study); the analysis is limited to genes with known metabolite interactions due
  to the unpaired nature of the datasets.
- When a gene of interest is not a discoverable gene in the HMDB/GWAS Catalog network
  objects (e.g. absent from discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData),
  since its node type cannot be traced.
tools:
- name: metGWAS 1.0
  role: Provides the GWAS Catalog network representation used to trace each intersection
    gene's position and assign primary vs first-neighbour node labels
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Runtime environment in which the metGWAS 1.0 scripts and .RData network objects
    are loaded and the node-type tabulation is performed
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:6f4115c5c76ecc231d4dcdccdd9dfd1264eefe2e7db36f282c96ae314a22ce34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolite-gene-intersection-analysis

## Summary

A metGWAS 1.0-based R procedure that classifies each gene in a metabolic/GWAS intersection set by its provenance in the GWAS Catalog network — either a primary keyword node or a first-neighbour node — to establish where the intersecting genes originate.

## When to use

Use when you have an intersection gene set from a metabolic-vs-genomic hypergeometric comparison and need to determine, gene by gene, whether each gene sits at a primary keyword node or a first-neighbour node in the GWAS subnetwork.

## When NOT to use

- When the goal is to generate or statistically test the metabolic vs genomic gene sets themselves — that upstream step is metGWAS 1.0's hypergeometric test, not this provenance classification.
- When the metabolite-gene associations are paired (matched from the same metabolomics-GWAS study); the analysis is limited to genes with known metabolite interactions due to the unpaired nature of the datasets.
- When a gene of interest is not a discoverable gene in the HMDB/GWAS Catalog network objects (e.g. absent from discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData), since its node type cannot be traced.

## Inputs

- GWASnetwork_discoverableGenesOnly_2021_10_14.RData — GWAS Catalog network object restricted to discoverable genes
- Intersection gene sets from metGWAS 1.0 hypergeometric comparison of metabolic and genomic gene sets (per case study)

## Outputs

- gene_node_classification.csv — per-gene, per-case-study table of node-type labels (primary keyword node vs first-neighbour node)

## How to apply

Load the GWAS Catalog network object GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB, downloaded from the Harvard Dataverse link in the README) into R to access the network representation of the GWAS Catalog. Load the case study intersection gene sets produced upstream by metGWAS 1.0's hypergeometric comparison of metabolic and genomic gene sets (e.g. the nine glycerophospholipid-pathway genes from case study 1: APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7). For each intersection gene, trace its position in the GWAS subnetwork within metGWAS 1.0 and label it as a primary keyword node (directly tied to the disease/phenotype search keyword) or a first-neighbour node (one hop away). Tabulate the node-type label per gene per case study and export to gene_node_classification.csv; interpret all-first-neighbour vs all-primary patterns as diagnostic of where the signal originates.

## Related tools

- **metGWAS 1.0** (Provides the GWAS Catalog network representation used to trace each intersection gene's position and assign primary vs first-neighbour node labels) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Runtime environment in which the metGWAS 1.0 scripts and .RData network objects are loaded and the node-type tabulation is performed) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript -e 'load("objects/GWASnetwork_discoverableGenesOnly_2021_10_14.RData"); source("metGWAS 1.0_19 April.R")'
```

## Evaluation signals

- Every gene in the intersection set must receive exactly one node-type label (primary keyword or first-neighbour) for each case study — no gene left unlabeled.
- For case study 1 (cardiovascular disease), all classified genes should come out as first-neighbour nodes; for case study 2 (urine/kidney), all as primary nodes.
- The nine case study 1 genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) must be traceable in GWASnetwork_discoverableGenesOnly_2021_10_14.RData before labeling.
- Output CSV schema contains gene identifier, node-type label, and case study identifier columns; row count equals intersection gene count.
- Environment setup must have completed via qvalue and bioconductor install.R and RSelenium Driver SetUp.R before running metGWAS 1.0_19 April.R.

## Limitations

- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516KB) is not included in the supplementary material; it must be downloaded from the Harvard Dataverse link, so offline execution is not possible.
- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the metabolomics and GWAS datasets.
- Case study 2's intersection genes showed only marginal significance (P = 0.10 and P = 0.13) upstream, so the resulting node-type labels rest on a weakly supported gene set.

## Evidence

- [methods] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [abstract] Genomic gene sets from GWAS Catalog network: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [results] Nine case study 1 genes: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7)"
- [readme] Unpaired-data limitation: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets,"
- [readme] R script for running the workflow: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
