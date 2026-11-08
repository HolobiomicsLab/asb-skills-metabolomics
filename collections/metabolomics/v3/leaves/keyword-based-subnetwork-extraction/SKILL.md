---
name: keyword-based-subnetwork-extraction
description: Use when you have a precomputed GWAS Catalog network object (e.g. GWASnetwork_discoverableGenesOnly
  RData) and need disease/trait-focused subnetworks around specific keyword-matched
  nodes for metGWAS 1.0 over-representation analysis.
when_to_use_negative:
- You need to build the GWAS network from raw GWAS Catalog records rather than reuse
  the precomputed discoverableGenesOnly network object
- Your trait of interest has no node matching the supplied keywords in the GWAS Catalog
  network (keyword filtering will yield no primary nodes)
- You require edges below the significance thresholds (Jaccard < 0.5 or FDR-corrected
  P > 0.05), which this workflow discards by design
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3295
tools:
- name: metGWAS 1.0
  role: R workflow that loads and operates on the GWAS Catalog network representation
    from which keyword subnetworks are extracted
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: R/Bioconductor package used to compute the FDR-corrected P values applied
    with the Jaccard >= 0.5 edge-significance threshold
- name: R
  role: Execution environment for the metGWAS 1.0 pipeline scripts (qvalue and bioconductor
    install.R, RSelenium Driver SetUp.R, metGWAS 1.0_19 April.R)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:2a192f34accbf5d379e197fb49c450bc504fd5e4d7544fa717a448f016f94077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# keyword-based-subnetwork-extraction

## Summary

Extracts disease/trait-specific GWAS subnetworks from the metGWAS 1.0 GWAS Catalog network representation by retaining only significant edges (Jaccard >= 0.5, FDR-corrected P <= 0.05), filtering nodes by trait keywords, and pulling out each keyword node plus its first-neighbour subnetwork.

## When to use

Use when you have a precomputed GWAS Catalog network object (e.g. GWASnetwork_discoverableGenesOnly RData) and need disease/trait-focused subnetworks around specific keyword-matched nodes for metGWAS 1.0 over-representation analysis.

## When NOT to use

- You need to build the GWAS network from raw GWAS Catalog records rather than reuse the precomputed discoverableGenesOnly network object
- Your trait of interest has no node matching the supplied keywords in the GWAS Catalog network (keyword filtering will yield no primary nodes)
- You require edges below the significance thresholds (Jaccard < 0.5 or FDR-corrected P > 0.05), which this workflow discards by design

## Inputs

- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516 KB) — network representation of the GWAS Catalog, downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP
- List of disease/trait keyword strings (e.g. 'Kidney', 'Urine') used to identify primary nodes

## Outputs

- gwas_keyword_subnetworks.RData containing keyword-filtered subnetwork objects (each a primary keyword node plus its first-neighbour nodes, with only Jaccard >= 0.5 / FDR <= 0.05 edges)

## How to apply

Load the GWASnetwork_discoverableGenesOnly_2021_10_14.RData object (Harvard Dataverse doi:10.7910/DVN/B32FRP) into R via metGWAS 1.0 and compute pairwise Jaccard coefficients between GWAS study gene sets in the network. Retain only network edges where the Jaccard coefficient is >= 0.5 and the FDR-corrected P value is <= 0.05, computing FDR correction with the qvalue package; this significance threshold defines which edges connect GWAS studies sharing trait gene sets. Filter the surviving network nodes by disease/trait keywords (e.g. 'Kidney', 'Urine') to designate primary nodes. For each primary node, extract the subnetwork comprising that node plus its first-neighbour nodes from the filtered network, and save all extracted subnetworks as gwas_keyword_subnetworks.RData.

## Related tools

- **metGWAS 1.0** (R workflow that loads and operates on the GWAS Catalog network representation from which keyword subnetworks are extracted) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (R/Bioconductor package used to compute the FDR-corrected P values applied with the Jaccard >= 0.5 edge-significance threshold)
- **R** (Execution environment for the metGWAS 1.0 pipeline scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R, metGWAS 1.0_19 April.R))

## Examples

```
load('GWASnetwork_discoverableGenesOnly_2021_10_14.RData'); subnets <- lapply(kidneyNodes, function(n) induced_subgraph(gwasNet, c(n, neighbors(gwasNet, n)))); save(subnets, file='gwas_keyword_subnetworks.RData')
```

## Evaluation signals

- The saved gwas_keyword_subnetworks.RData file exists and contains subnetwork objects
- Every edge in each extracted subnetwork satisfies Jaccard >= 0.5 and FDR-corrected P <= 0.05
- Each extracted subnetwork consists of a primary keyword-matched node plus its first-neighbour nodes
- Primary nodes match the user-supplied disease/trait keywords (e.g. 'Kidney', 'Urine')
- The loaded network matches the deposited GWASnetwork_discoverableGenesOnly_2021_10_14.RData object from doi:10.7910/DVN/B32FRP

## Limitations

- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516 KB) is too large for the supplement and is only available via an external Harvard Dataverse download link
- The analysis is limited to genes with known metabolite interactions ('discoverableGenesOnly') because of the unpaired nature of the metabolomics and GWAS data sets
- Subnetwork scope is fixed at first neighbours of keyword-matched primary nodes; deeper neighbourhoods are not extracted
- No changelog found for the repository, so script version drift is possible

## Evidence

- [methods] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [discussion] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [readme] Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
