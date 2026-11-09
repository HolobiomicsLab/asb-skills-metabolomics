---
name: gwas-network-node-classification
description: Use when you have metGWAS 1.0 intersection gene sets from a hypergeometric
  comparison of metabolic and genomic gene sets and need to trace whether each gene
  is a primary keyword node or a first-neighbour node in the GWAS subnetwork.
when_to_use_negative:
- You only need pathway-level over-representation p-values from the hypergeometric
  test, not node-level provenance of the intersecting genes.
- Your gene sets come from paired metabolomics-GWAS data rather than the standalone,
  unpaired metabolomics + GWAS Catalog setting metGWAS 1.0 is designed for.
- The GWAS Catalog network object has not been loaded (GWASnetwork_discoverableGenesOnly_2021_10_14.RData
  is missing), since node positions cannot be traced without it.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3366
tools:
- name: metGWAS 1.0
  role: R workflow used to trace each intersection gene's position in the GWAS Catalog
    subnetwork and label it as primary keyword node or first-neighbour node
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Runtime environment in which the GWASnetwork RData object and metGWAS 1.0
    scripts are executed for node classification
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:f34e62a3df42a0fa335ad20db7af0b24a2ed7b3dc05cd9f572a294bbee9fce0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gwas-network-node-classification

## Summary

Classify each gene in a metGWAS 1.0 intersection gene set by its position in the GWAS Catalog network representation — primary keyword node versus first-neighbour node — to determine whether metabolic/GWAS overlaps originate directly from keyword-matched genes or from their network neighbours.

## When to use

Use when you have metGWAS 1.0 intersection gene sets from a hypergeometric comparison of metabolic and genomic gene sets and need to trace whether each gene is a primary keyword node or a first-neighbour node in the GWAS subnetwork.

## When NOT to use

- You only need pathway-level over-representation p-values from the hypergeometric test, not node-level provenance of the intersecting genes.
- Your gene sets come from paired metabolomics-GWAS data rather than the standalone, unpaired metabolomics + GWAS Catalog setting metGWAS 1.0 is designed for.
- The GWAS Catalog network object has not been loaded (GWASnetwork_discoverableGenesOnly_2021_10_14.RData is missing), since node positions cannot be traced without it.

## Inputs

- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (GWAS Catalog network object, 64,516 KB, from Harvard Dataverse doi:10.7910/DVN/B32FRP)
- Intersection gene sets from metGWAS 1.0 hypergeometric comparison (e.g. case study 1 cardiovascular genes, case study 2 kidney-related genes)
- metGWAS 1.0 R scripts (metGWAS 1.0_19 April.R) with the objects folder containing discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData

## Outputs

- gene_node_classification.csv (node-type label — primary keyword node vs first-neighbour node — per intersection gene per case study)

## How to apply

Load the GWAS Catalog network object GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516 KB, downloaded from the Harvard Dataverse link in the metGWAS 1.0 README) into R to access the network representation comprising 100s of GWAS studies. Load the case-study intersection gene sets produced upstream by metGWAS 1.0's hypergeometric comparison of metabolic and genomic gene sets. For each intersection gene, use metGWAS 1.0 in R to trace its position in the GWAS subnetwork and label it as a primary keyword node (directly matched to a disease/phenotype keyword) or a first-neighbour node (adjacent to a keyword node). Tabulate the node-type label per gene per case study and write the result to gene_node_classification.csv. In the published evaluation, case study 1 (cardiovascular disease, e.g. the nine genes APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7 in the KEGG glycerophospholipid metabolism pathway) yielded intersections derived exclusively from first-neighbour nodes, while case study 2 (urine metabolome / kidney) yielded intersections derived exclusively from primary nodes — so a correct run resolves every gene into exactly one of the two node types.

## Related tools

- **metGWAS 1.0** (R workflow used to trace each intersection gene's position in the GWAS Catalog subnetwork and label it as primary keyword node or first-neighbour node) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Runtime environment in which the GWASnetwork RData object and metGWAS 1.0 scripts are executed for node classification)

## Examples

```
Rscript metGWAS_1.0_19_April.R  # after loading GWASnetwork_discoverableGenesOnly_2021_10_14.RData to trace intersection genes' node types
```

## Evaluation signals

- Every intersection gene receives exactly one label: primary keyword node or first-neighbour node; no gene is unlabeled or double-labeled.
- Case study 1 (cardiovascular) intersection genes classify as first-neighbour nodes only, matching the published finding.
- Case study 2 (urine/kidney) intersection genes classify as primary nodes only, matching the published finding.
- gene_node_classification.csv contains one row per intersection gene per case study with a node-type column.
- The network object loaded matches GWASnetwork_discoverableGenesOnly_2021_10_14.RData downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP.

## Limitations

- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516 KB) is too large for the repository and must be downloaded from an external Harvard Dataverse link.
- Analysis is limited to genes with known metabolite interactions due to the unpaired nature of the standalone metabolomics and GWAS data sets.
- Classification applies to the GWAS Catalog network representation; results depend on that specific 2021-10-14 snapshot.

## Evidence

- [methods] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [other] The intersecting genes of case study 1 were derived exclusively from first-neighbour nodes, while the intersecting genes of case study 2 were derived exclusively from primary nodes.: [evidence span withheld — anchor status: not_found]
- [readme] GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB): "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
